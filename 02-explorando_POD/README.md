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
  name: meu-pod
spec:
  containers:
    - name: app
      image: nginx
      ports:
        - containerPort: 80
```
ou
```bash
kubectl run meu-pod --port=80 --image=nginx --dry-run=client -o yaml > simple-pod.yaml
```
Crie o Pod:

```bash
kubectl apply -f simple-pod.yaml
```


## 2️ Explorar a estrutura do Pod
Use os comandos abaixo p`bash
Copiar
Editar
kubectl exec -it meu-pod -- /bin/bash
Testar conectividade interna:

```bash
apt update && apt install curl -y
curl localhost
```

## 4️. Criar um Pod com volume compartilhado
Edite ou crie um novo arquivo chamado pod-com-volume.yaml:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-volume
spec:
  volumes:
    - name: dados-compartilhados
      emptyDir: {}
  containers:
    - name: writer
      image: busybox
      command: [ "sh", "-c", "echo 'Olá do container writer!' > /dados/mensagem.txt && sleep 3600" ]
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
kubectl apply -f pod-com-volume.yaml
```

Acesse o container reader e leia o arquivo gravado pelo writer:
```bash
kubectl exec -it pod-com-volume -c reader -- cat /dados/mensagem.txt
```
## 5️. Criar um Pod com variáveis de ambiente
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

## 6️. Criar um Pod com múltiplos containers (sidecar)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-sidecar
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - containerPort: 80
    - name: log-agent
      image: busybox
      command: ["sh", "-c", "while true; do echo 'Log do sidecar' >> /var/log/web.log; sleep 5; done"]
```

Acompanhe os logs dos containers:

```bash
kubectl logs pod-com-sidecar -c log-agent -f
```

## ✅ Conclusão
Neste exercício, você explorou:

- Criação e estrutura básica de um Pod
- Acesso ao shell e teste interno
- Uso de volumes emptyDir
- Definição e uso de variáveis de ambiente
- Múltiplos containers (sidecar)
- Este exercício é excelente para entender a base de toda aplicação no Kubernetes: o Pod.
