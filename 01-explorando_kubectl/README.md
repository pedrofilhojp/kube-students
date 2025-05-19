# 🧪 Exercício: Deploy de uma Aplicação no Kubernetes com Kind
Imagem usada: gcr.io/google-samples/kubernetes-bootcamp

Este exercício foi desenvolvido para ser executado em um cluster Kubernetes criado com Kind. Antes de começar, certifique-se de ter o Kind e kubectl instalados.

## 1️⃣ Criar o Deployment via linha de comando
Use o comando abaixo para criar um deployment com a imagem kubernetes-bootcamp:

```bash
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```
Esse comando cria um Deployment chamado kubernetes-bootcamp com um Pod rodando a imagem especificada.

## 2️⃣ Observar os componentes criados
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
Descrever os detalhes de um Pod:

```bash
kubectl describe pod <nome-do-pod>
```
## 3️⃣ Expor a aplicação na porta 8080
Crie um Service para expor a aplicação:

```bash
kubectl expose deployment kubernetes-bootcamp --type=NodePort --port=8080 --target-port=8080
```
Isso cria um Service do tipo NodePort que encaminha a porta 8080 para o Pod.

## 4️⃣ Como acessar a aplicação
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
## 5️⃣ Escalar a aplicação para 5 réplicas
Para escalar o número de réplicas do deployment para 5:

```bash
kubectl scale deployment kubernetes-bootcamp --replicas=5
```
Verifique o resultado:

```bash
kubectl get pods
```
Você deverá ver 5 Pods rodando.

## 6️⃣ Acessar o shell do container
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

Esse passo a passo proporciona uma introdução prática ao uso básico de Kubernetes com foco em operações com kubectl, observação de recursos e interação com Pods. Ideal para quem está começando com Kubernetes em ambiente local via Kind