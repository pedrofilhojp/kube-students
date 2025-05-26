# 🧪 Exercício Completo: ReplicaSet, Deployment e Estratégias de Deploy no Kubernetes
Este exercício foi desenvolvido para ambientes locais com Kind e visa explorar:

- Criação manual de ReplicaSet
- Deploy com Deployment via YAML
- Escalonamento de réplicas
- Estratégias de atualização (RollingUpdate e Recreate)
- Rollback de versões

## 🎯 Objetivos
- Criar e entender ReplicaSet
- Criar e atualizar Deployments com controle de versão
- Explorar estratégias de atualização
- Realizar rollback em caso de falha

## 🔧 1. Criando um ReplicaSet
Crie o arquivo replicaset.yaml:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: hello-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-app-pod
  template:
    metadata:
      labels:
        app: hello-app-pod
    spec:
      containers:
        - name: hello-container
          image: busybox
          command: ["sh", "-c", "while true; do echo 'Hello from ReplicaSet'; sleep 10; done"]
```

Aplique:

```bash
kubectl apply -f replicaset.yaml
kubectl get pods -l app=hello-app
```

Teste de auto-recuperação:

```bash
kubectl delete pod <nome-do-pod>
```

O ReplicaSet criará automaticamente um novo Pod para manter o número de réplicas.

## 🚀 2. Criando um Deployment com estratégia RollingUpdate
Crie o arquivo deployment.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-container
          image: nginx:1.21
          ports:
            - containerPort: 80
Aplicar:

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
```
## 📘 Explicação: Estratégia RollingUpdate
Esta é a estratégia **padrão** do Kubernetes.

- Substitui os Pods gradualmente, um a um.
- Garante alta disponibilidade, pois os novos Pods são criados antes que os antigos sejam removidos.
- Ideal para sistemas que não podem ter downtime durante atualizações.

Parâmetros (Estas opções são opcionais):

- **maxUnavailable**: quantos Pods podem estar fora do ar durante o rollout.
- **maxSurge**: quantos Pods extras (além do número desejado) podem ser criados temporariamente.

## 🔄 3. Atualizando com controle de rollout
Atualize a imagem:

```bash
kubectl set image deployment/web-deployment web-container=nginx:1.25
kubectl rollout status deployment/web-deployment
```
Veja o histórico:

```bash
kubectl rollout history deployment/web-deployment
🧯 4. Rollback após uma falha
```
Atualize para uma imagem inválida:

```bash
kubectl set image deployment/web-deployment web-container=nginx:erro
```
Acompanhe a falha e faça rollback:

```bash
kubectl rollout undo deployment/web-deployment
```
## 🔁 5. Criando um Deployment com estratégia Recreate
Crie o arquivo deployment-recreate.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-recreate
spec:
  replicas: 2
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: recreate-app
  template:
    metadata:
      labels:
        app: recreate-app
    spec:
      containers:
        - name: app-container
          image: nginx:1.21
          ports:
            - containerPort: 80
```
Aplique:

```bash
kubectl apply -f deployment-recreate.yaml
```
Atualize a imagem:

```bash
kubectl set image deployment/app-recreate app-container=nginx:1.25
kubectl rollout status deployment/app-recreate
```
## 📘 Explicação: Estratégia Recreate
- Remove todos os Pods antigos antes de criar os novos.
- Pode gerar tempo de inatividade (downtime).
- Indicado para aplicações com estado global ou que não suportam múltiplas versões em paralelo (ex: bancos de dados que requerem lock exclusivo).

## 🔍 6. Verificação e limpeza
Listar objetos criados:

```bash
kubectl get deployments
kubectl get pods
kubectl get rs
```
Remover:

```bash
kubectl delete deployment web-deployment app-recreate
kubectl delete replicaset hello-rs
```
## ✅ Conclusão
Neste exercício você:

- Criou ReplicaSet manualmente e compreendeu sua função de manter réplicas
- Criou Deployments com diferentes estratégias de rollout:
- - RollingUpdate: alta disponibilidade, atualização contínua
- - Recreate: substituição completa, com possível downtime
- Realizou atualizações e rollbacks de maneira segura

Com isso, você domina o ciclo de vida de aplicações no Kubernetes com foco em disponibilidade, controle e automação
