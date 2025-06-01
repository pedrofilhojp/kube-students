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

## Motivação
Um POD sozinho não é suficiente para colocar uma aplicação no AR. Quando o POD cai ou ocorre um erro na aplicação, simplementes a aplicação torna-se indisponível.

Há várias situações que podem ocorrer na aplicação que necessita de uma intervenção automática do cluster. O kubernetes tem vários recursos extras para apoiar a disponibilidade de aplicação, as principais são:
- Replicaset
- Deployment
- Services
- HPA e VPA

Aqui, veremos por enquanto apenas o **Replicaset** e **Deployment**

## 🔧 1. Criando um ReplicaSet
Um ReplicaSet é um recurso no Kubernetes que garante a disponibilidade de um número específico de réplicas (pods) em execução. Ele monitora os pods e, se algum falhar ou for deletado, o ReplicaSet cria um novo pod automaticamente para manter o estado desejado.

✔️ Principais motivos para usar um ReplicaSet:

- Garante que sempre exista a quantidade correta de pods em execução.
- Recupera automaticamente pods que falham.
- Permite escalar a aplicação facilmente (aumentar ou diminuir réplicas).

Crie o arquivo rs-simples.yaml:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-simples
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod
  template:
    metadata:
      labels:
        app: pod
    spec:
      containers:
        - name: hello-container
          image: busybox
          command: ["sh", "-c", "while true; do echo 'Hello from ReplicaSet'; sleep 10; done"]
```

Aplique:

```bash
kubectl apply -f rs-simples.yaml
kubectl get rs
kubectl get pods -l app=app
```

Teste de auto-recuperação:

```bash
kubectl delete pod <nome-do-pod>
```

O ReplicaSet criará automaticamente um novo Pod para manter o número de réplicas.

### Limitações do ReplicaSet
Embora o ReplicaSet mantenha as réplicas, ele não gerencia atualizações da aplicação (por exemplo, quando você troca a imagem do contêiner para uma nova versão). Se você quiser implantar uma nova versão da aplicação, precisa **deletar e criar** um novo ReplicaSet manualmente.

## 🚀 2. Criando um Deployment 

O **Deployment** é um recurso de nível mais alto que gerencia **ReplicaSets** para você. Ele permite:

- Declarar o estado desejado (imagem, número de réplicas, etc.) de maneira declarativa.
- Executar atualizações sem downtime (com estratégias como Rolling Update).
- Reverter facilmente para versões anteriores em caso de problemas.
- Histórico de revisões para rastrear mudanças ao longo do tempo.

Crie o arquivo deploy-web.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-web
spec:
  replicas: 2
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
```

Aplicar os comandos:

```bash
kubectl apply -f deploy-web.yaml
kubectl get deployments
```
## 📘 Criando um Deployment com strategy RollingUpdate

Edite o arquivo "deploy-web.yaml" adicionando a sessão "**strategy**".

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-web
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
```

Em seguinte aplique as alterações
```bash
kubectl apply -f deploy-web.yaml
kubectl get deployments
```

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
Atualize para uma imagem **INVÁLIDA**:

```bash
kubectl set image deployment/web-deployment web-container=nginx:erro
```
Acompanhe a falha e faça rollback:

```bash
kubectl rollout undo deployment/web-deployment
```
## 🔁 5. Criando um Deployment com strategy Recreate
Crie o arquivo deployment-recreate.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-recreate
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
kubectl apply -f deploy-recreate.yaml
```
Atualize a imagem:

```bash
kubectl set image deployment/deploy-recreate=nginx:1.25
kubectl rollout status deployment/deploy-recreate
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
kubectl delete deployment deploy-web deploy-recreate
kubectl delete replicaset rs-simples
```
## ✅ Conclusão
Neste exercício você:

- Criou ReplicaSet manualmente e compreendeu sua função de manter réplicas
- Criou Deployments com diferentes estratégias de rollout:
- - RollingUpdate: alta disponibilidade, atualização contínua
- - Recreate: substituição completa, com possível downtime
- Realizou atualizações e rollbacks de maneira segura

Com isso, você domina o ciclo de vida de aplicações no Kubernetes com foco em disponibilidade, controle e automação
