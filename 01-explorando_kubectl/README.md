# 🧪 Exercício: Deploy de uma Aplicação no Kubernetes com Kind
Imagem usada: gcr.io/google-samples/kubernetes-bootcamp

Este exercício foi desenvolvido para ser executado em um cluster Kubernetes criado com Kind. Antes de começar, certifique-se de ter o Kind e kubectl instalados.

## 1️. Criar o Deployment via linha de comando
Use o comando abaixo para criar um deployment com a imagem kubernetes-bootcamp:

```bash
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```
Esse comando cria um Deployment chamado kubernetes-bootcamp com um Pod rodando a imagem especificada.

## 2️. Observar os componentes criados
Você pode explorar os componentes criados com os comandos:

Verificar o Deployment:

```bash
kubectl get deployments
```
Verificar o ReplicaSet criado pelo Deployment:

```bash
kubectl get rs
```
Verificar os Pods em execução:

```bash
kubectl get pods
```
Visualizar todos os componentes anteriores
```bash
kubectl get all
```

Descrever os detalhes de um Pod:

```bash
kubectl describe pod <nome-do-pod>
```
## 3. Usando Labels

Observamos que foi criado um Deployment automaticamente, vamos buscar os detalhes deste componente com o comando abaixo:
```bash
 kubectl describe deployment kubernetes-bootcamp 
```
Observe que o POD foi selecionado a partir da label **app=kubernetes-bootcamp**

Vamos selecionar esse POD usando a label

```bash
kubectl get pods -l app=kubernetes-bootcamp
```
Você pode fazer o mesmo com o service
```bash
kubectl get services -l app=kubernetes-bootcamp
```
Também podemos adicionar novas Labels ao POD 
```bash
export POD_NAME="$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')"
echo "Name of the Pod: $POD_NAME"
```
```bash
kubectl label pods "$POD_NAME" version=v1
```
Observe que foi adicionado uma nova Label ao POD
```bash
kubectl describe pods "$POD_NAME"
```

## 4. Expor a aplicação na porta 8080
Crie um Service para expor a aplicação:

```bash
kubectl expose deployment kubernetes-bootcamp --type=NodePort --port=8080 --target-port=8080
```
Isso cria um Service do tipo NodePort que encaminha a porta 8080 para o Pod.

## 5. Como acessar a aplicação
Para acessar a aplicação em um cluster Kind:

Descubra a porta mapeada pelo NodePort:

```bash
kubectl get service kubernetes-bootcamp
```
Você verá algo como:

```pgsql
NAME                  TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes-bootcamp   NodePort   10.96.XXX.XXX  <none>        8080:<NODEPORT>  Xm
```
Obtenha o IP do container do Kind:

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' kind-control-plane
```
Acesse a aplicação no navegador ou via curl:

```bash
curl http://<IP_KIND>:<NODEPORT>
```
## 6. Escalar a aplicação para 5 réplicas
Para escalar o número de réplicas do deployment para 5:

```bash
kubectl scale deployment kubernetes-bootcamp --replicas=5
```
Verifique o resultado:

```bash
kubectl get pods
```
Você deverá ver 5 Pods rodando.

## 7. Acessar o shell do container
Para acessar o shell interativo dentro de um dos Pods:

Liste os Pods:

```bash
kubectl get pods
```
Use o nome de um dos Pods listados:

```bash
kubectl exec -it <nome-do-pod> -- /bin/bash
```
Você estará dentro do container e poderá executar comandos como curl localhost:8080.

✅ Fim do exercício.

Esse passo a passo proporciona uma introdução prática ao uso básico de Kubernetes com foco em operações com kubectl, observação de recursos e interação com Pods.