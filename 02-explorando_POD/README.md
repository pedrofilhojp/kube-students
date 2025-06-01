# 🧪 Exercício: Explorando Pods no Kubernetes
Este exercício é voltado ao entendimento profundo do que é um Pod, suas características, e como interagir diretamente com ele.

## 🎯 Objetivo
- Criar Pods manualmente
- Entender a estrutura de um Pod
- Interagir com containers dentro do Pod
- Montar volumes
- Rodar múltiplos containers no mesmo Pod (sidecar pattern)

## 1️. Criar um Pod a partir de um manifest YAML
Crie um arquivo chamado simple-pod.yaml com o conteúdo abaixo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-simples
spec:
  containers:
    - name: app
      image: nginx
      ports:
        - containerPort: 80
```
```bash
kubectl apply -f simple-pod.yaml
```

ou
```bash
kubectl run pod-simples --port=80 --image=nginx --dry-run=client -o yaml > pod-simples.yaml
```

## 2. Visualizando o pod criado

Sempre que desejar visualizar os PODs criados, execute
```bash
kubectl get pods
```

Também é possível explorar a estrutura do POD após criado

```bash
kubectl describe pod/pod-simples
```
Após executar o describe, uma das sessões mais importantes é a "**Events**". Local onde é registrado tudo que aconteceu desde a criação do POD, então, é um ótimo lugar para iniciar um **Throubleshot**, ou seja, resolução de problemas.

## 3. Explorar a estrutura do Pod
O comando a seguir é utilizado para acessar o shell do POD.

```bash
kubectl exec -it pod-simples -- /bin/bash
```
Obs. Há containers que são projetados para não ter shell, ou utiliza outros tipos de shell diferente do bash, como exemplo **ksh**.

Após acessar o container, execute os comandos a seguir para testar conectividade interna:

```bash
apt update && apt install curl -y
curl localhost
```

## 4. Criar um Pod com volume compartilhado
Neste exemplo, iremo criar um POD com dois containers (essa técnica chama-se **sidecar**) e um volume compartilhado entre os containers.

O primeiro container chama-se **write**, ele irá escrever no arquivo /dados/messagem.txt um texto.

Podemos verificar esse conteúdo no container **reader** e verificar seu conteúdo.

Edite ou crie um novo arquivo chamado pod-com-volume.yaml:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-compartilhado
spec:
  volumes:
    - name: dados-compartilhados
      emptyDir: {}
  containers:
    - name: writer
      image: busybox
      command: [ "sh", "-c", "echo 'Olá do container writer!' > /dados/menssagem.txt && sleep 3600" ]
      volumeMounts:
        - name: dados-compartilhados
          mountPath: /dados
    - name: reader
      image: busybox
      command: [ "sh", "-c", "sleep 3600" ]
      volumeMounts:
        - name: dados-compartilhados
          mountPath: /dados
```

Crie o Pod:
```bash
kubectl apply -f pod-volume-compartilhado.yaml
```

Acesse o container **reader** e leia o arquivo gravado pelo **writer**:
```bash
kubectl exec -it pod-volume-compartilhado -c reader -- cat /dados/menssagem.txt
```


## 5. Outro exemplo de **sidecar**

Este exemplo reflete uma situação muito comum em ambiente Devops, na qual temos o container principal da aplicação (neste caso o **nginx**), e outro container apenas para ler os logs e enviá-los para algum sistema de gerencimamento centralizado de logs (estamos representado a aplicação de ler logs com o container **busybox**).


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx-sidecar
spec:
  volumes:
    - name: logs-compartilhado
      emptyDir: {}
  containers:
    - name: web
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - name: logs-compartilhado
          mountPath: /var/log/nginx
    - name: log-agent
      image: busybox
      volumeMounts:
        - name: logs-compartilhado
          mountPath: /var/log/nginx
      command: ["sh", "-c", "tail -f /var/log/nginx/access.log;"]
```

Acompanhe os logs do **web** pelo container **log-agent**:

```bash
kubectl logs pod-nginx-sidecar -c log-agent -f
```

### 5.1. Port-forward

Utilizamos este recurso para acessar algum POD com intuito de testar, debugar algum problema na aplicação, entre outros.

Em outro terminal abra uma comunicação do seu PC com o container **web** usando um portforward

```bash
kubectl port-forward pods/pod-nginx-sidecar 9095:80
```

Basta acessar pelo navegador ou curl o url http://127.0.0.1:9095 e visualizar a leitura dos logs pelo container do **log-agent**

## 6. Criar um Pod com variáveis de ambiente

Criar um POD com passando via parâmetro variáveis ambiente é uma prática muito comum (praticamente 99% dos caso), para configuramos uma aplicação em formato de container. 

A seguir, usamos a configuração **env** na definição do container para setar/configurar/alterar alguma variável ambiente durante inicialização do container/aplicação.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-env
spec:
  containers:
    - name: env-demo
      image: busybox
      command: [ "sh", "-c", "echo VAR1=$VAR1 && sleep 3600" ]
      env:
        - name: VAR1
          value: "Exemplo de variável"
```

```bash
kubectl apply -f pod-com-env.yaml
kubectl logs pod-com-env
```

## 7. Excluíndo o que fizemos
Podemos deletar os POD especificando-os diretamente
```bash
kubectl delete pod pod-simples pod-volume-compartilhado pod-nginx-sidecar pod-com-env
```

Ou invocando os arquivos .yml
```bash
kubectl delete -f .
```
O comando acima precisa que vc esteja no mesmo diretório dos arquivos .yaml

## ✅ Conclusão
Neste exercício, você explorou:

- Criação e estrutura básica de um Pod
- Acesso ao shell e teste interno
- Uso de volumes emptyDir
- Múltiplos containers (sidecar)
- Acesso ao POD via port-forward
- Definição e uso de variáveis de ambiente
- Este exercício é excelente para entender a base de toda aplicação no Kubernetes: o Pod.
