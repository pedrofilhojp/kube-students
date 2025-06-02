# 🚀 Exercício: Explorando os tipos de Services no Kubernetes

Neste exercício, vamos entender os diferentes tipos de Services disponíveis no Kubernetes e como utilizá-los para expor aplicações — usando o **Nginx** como exemplo. Ao final, você será capaz de criar e diferenciar cada tipo de Service.

Para mais informações, utilize a documentação do Kubernetes, [Clique aqui](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/)

## Ajustes no kind

Para alguns exemplos de services, ficará melhor a compereensão se criarmos nosso cluster com vários workers. Logo, utilize o exemplo a seguir para criar o cluster kubernetes no kind

Crie um arquivo de nome "kind-config.yaml"

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Execute:

```bash
kind create cluster --config kind-config.yaml
```

---

Antes de iniciar, e de forma bem simples o objetiva, vamos compreender que um **Service** é um recurso de LoadBalance com capacidade de selecionar os PODs de uma aplicação dinamicamente. Ou seja, se uma aplicação que tinha 5 réplicas aumentou para 10, o Service consegue selecinar as novas replicas e balancear o trafego de rede também para elas.

Há vários algorítmos que se pode utilizar em um Loadbalance, mas o Service trabalha apenas com ***Round-Robin***. 

Há apenas 4 tipos de Services no kubernetes:

- **ClusterIP**: IP dentro do Cluster. Utilizado para expor as aplicações apenas dentro do cluster, para uma aplicação acessar outra;
- **NodePort**: Expor a aplicação para fora do cluster em ambiente **NÃO CLOUD**, privada ou pública;
- **LoadBalance**: O mesmo que ClusterIP, porém para ambientes de Cloud, isso quer dizer que, na AWS por exemplo, você utilizará o LoadBalance para expor a aplicação e você acessá-la;
- **ExternalName**: Mapear algum registro DNS externo para algum service interno no Cluster.

**Porque acessar a aplicação pelo SERVICE e não diretamente o POD?**

Para acessar a aplicação, precisamos do IP do POD, o problema é que, quando o POD cai ele não sobe com o mesmo IP. Além disso, a aplicação pode necessitar de várias réplicas para suportar a carga de acesso dos usuários.

O Service também sobe um endereço IP exclusivo para ele, mas, esse IP não é alterado, logo, se o POD cair e subir novamente com outro IP, o service consegue identificar o IP do novo POD e não terá problemas no acesso.

## 1️⃣ Service ClusterIP

O tipo **ClusterIP** é o padrão ao criar um Service no Kubernetes. Ele cria um IP interno acessível apenas dentro do cluster, permitindo a comunicação entre pods e outros recursos internos, mas **não** expondo o Service externamente.

**Limitações:**
- Não pode ser acessado fora do cluster.
- Ideal para aplicações internas, como backends ou bancos de dados.

**Criação do Service ClusterIP para Nginx:**

No exemplo a seguir, criamos ao mesmo tempo:
- 1 Deployment com 2 containers (1º nginx e 2º para alterar o conteúdo do arquivo index.html);
- 1 Service para os PODs criados pelo Deployment
- 1 POD isolado para realizar requisições http para os PODs através do service

Observer no arquivo, que cada item descrito acima, é separado por 3 traços (---). Para o modelo yaml, é como se indicarmos um novo elemento no arquivo.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      - name: ip-writer
        image: busybox
        command: ["/bin/sh", "-c"]
        # Cria um index.html com o IP da réplica antes do Nginx iniciar
        args:
          - |
            echo "<html><body><h1>Meu IP é: $(hostname -i)</h1></body></html>" > /html/index.html;
            sleep 3600;
        volumeMounts:
        - name: html
          mountPath: /html
      volumes:
      - name: html
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-geturl
spec:
  containers:
  - name: curl
    image: nginx
    command: ["/bin/sh", "-c"]
    args:
      - |
        while true; do
          echo "Fazendo requisição ao svc-clusterip:80";
          curl -s svc-clusterip | sed -n 's/.*<h1>\(.*\)<\/h1>.*/\1/p';
          echo "---";
          sleep 2;
        done
```



Para testar o service, vamos observar os logs do POD (pod-geturl)

```bash
kubectl apply -f svc-clusterip.yaml
kubectl logs pods/pod-geturl -f
```

Anote os endereços IPs que os PODs responderam.

Em outro terminal, delete algum dos POD... Após o POD entrar no estado de Running, observe que o Service já identificou o novo POD que foi criado, e mostrará nas requisições do POD (pod-geturl)

## 2️⃣ NodePort

O tipo NodePort expõe o Service em uma porta específica de cada nó do cluster. Assim, o Service pode ser acessado externamente pelo IP de qualquer nó e pelo NodePort atribuído (geralmente na faixa 30000-32767).

**Limitações:**

- Cada Service usa uma porta fixa do nó, que pode causar conflitos se não for bem gerenciado.
- Menos eficiente e seguro que LoadBalancer para ambientes de produção.

**2.1. Identificando os PODs**

Podemos utilizar os mesmos PODs da sessão anterior. Então, vamos primeiro identificar como está a distribuíção dos PODs entre os servidores do cluster:

```bash
kubectl get pods -o wide
```
```
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
deploy-nginx-8465c64cc6-chq8m   2/2     Running   0          100s    10.244.2.4   kind-worker2   <none>           <none>
deploy-nginx-8465c64cc6-jbjc7   2/2     Running   0          100s    10.244.2.5   kind-worker2   <none>           <none>
deploy-nginx-8465c64cc6-qrwh9   2/2     Running   0          5m32s   10.244.1.2   kind-worker    <none>           <none>
pod-geturl                      1/1     Running   0          5m32s   10.244.1.3   kind-worker    <none>           <none>

```
Observe que alguns foram criados no node **kind-worker** e **kind-worker2**

**2.2. Criação do Service NodePort**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort
```
```bash
kubectl get services
```

Observe que forçamos um mapeamento em TODOS os nodes (servidores Worker ou ControlPlane) com a porta 30080. Então, basta pegar o endereço IP de qualquer Node do cluster, e acessar com essa porta.
> **Observação:** Se não especificar a porta em `nodePort` o kubernetes escolherá alguma porta dentro do intervalo "30000-32767"
Esse service também faz um mapeamento interno no cluster, isso quer dizer que, é possível acessar esse Service de dentro do Cluster pela porta 80, `port: 80` 

```bash
kubectl get nodes -o wide
```

Acesse o Nginx em:

```cpp
http://<IP_DO_NO>:30080
```
## 3️⃣ LoadBalancer

O tipo LoadBalancer cria um balanceador de carga externo (quando disponível no provedor de nuvem) e direciona o tráfego para os pods. É ideal para aplicações expostas publicamente, como sites ou APIs.

**Limitações:**

- Depende do suporte do provedor de nuvem (não funciona em clusters locais sem um load balancer externo configurado).

- Visto que utiliza um recurso externo de LoadBalance disponível no provedor de nuvem, pode gerar custos adicionais.

**Criação do Service LoadBalancer**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-loadbalancer
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```
O IP externo é atribuído automaticamente:

```bash
kubectl get svc svc-loadbalancer
```

## 4️⃣ ExternalName

O tipo ExternalName permite expor um Service para um nome DNS externo. Ele não cria um proxy nem expõe um IP dentro do cluster. Apenas fornece um alias DNS para um recurso externo.

> **Observação**: Este recurso é pouco utilizado, mas pode ser extremamente útil quando há uma planejamento de preparar a aplicação no uso de um recurso que está fora do cluster, mas será futuramente, colocado para dentro do cluster.

**Limitações:**

- Não faz roteamento interno de tráfego.

- Apenas um mapeamento de nome DNS — não gera pods ou endpoints.

**Exemplo de Service ExternalName apontando para um servidor externo:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-external
spec:
  type: ExternalName
  externalName: google.com
```

Neste caso, qualquer requisição de dentro do Cluster para **svc-external.default.svc.cluster.local** será redirecionada para **google.com**.

##  Resumo das diferenças

| Tipo de Service   | Acessível de fora do cluster? | Load balancing?      | Usos típicos                 |
|--------------------|------------------------------|----------------------|------------------------------|
| **ClusterIP**      | ❌                            | ✅ (interno)         | Comunicação interna          |
| **NodePort**       | ✅ (pelo IP do nó)           | ✅ (básico)          | Testes, exposições simples   |
| **LoadBalancer**   | ✅ (público, se suportado)   | ✅ (com Load Balancer) | Produção em nuvem            |
| **ExternalName**   | ✅ (via DNS)                 | ❌                    | Alias DNS para recursos externos |


## 🚀 Importância dos "Endpoints" para o "Service"

Um **endpoint** em Kubernetes representa um conjunto de endereços IPs e portas onde um Service pode enviar tráfego.

Mais tecnicamente:

- Ele **mapeia** o Service para os Pods (ou outros backends) que realmente respondem pelas requisições.

**🛠️ Como funciona?**
- Quando você cria um Service (ClusterIP, NodePort, etc.), o Kubernetes automaticamente cria um recurso de tipo **Endpoints** com o mesmo nome do Service.
- O Endpoints lista os Pods prontos (com Ready = True) que correspondem ao seletor (**selector**) do Service.
- O kube-proxy e outras ferramentas usam o Endpoints para enviar o tráfego para os Pods correto

**🔎 Exemplo**

Suponha que você tenha um Service chamado **svc-clusterip** que seleciona pods com **app=nginx**.

Veja os endpoints com:
```bash
kubectl get endpoints svc-clusterip
```

Exemplo de saída:

```yaml
NAME               ENDPOINTS              AGE
nginx-clusterip    10.244.0.10:80,10.244.0.11:80,10.244.0.12:80   10m
```
Isso significa:
- O Service nginx-clusterip tem 3 endpoints ativos (os IPs e portas dos Pods prontos para receber requisições).

Pontos importantes:
- Endpoints são dinâmicos! Se um Pod morre ou fica NotReady, o Kubernetes atualiza automaticamente os Endpoints.
- Pod IP = Endpoint: O IP real do Pod que responde pela aplicação.
- Para acessar os Endpoints diretamente (por exemplo, sem Service), você teria que usar os IPs internos do cluster — não recomendado em produção.

**✏️ Em resumo**

| Termo        | O que é?                                                        |
|--------------|------------------------------------------------------------------|
| **Service**  | Abstração de rede, tem IP virtual e política de acesso.         |
| **Endpoints**| Lista de IP:porta dos Pods que **realmente** recebem o tráfego. |


## 📝 Tarefas
✅ 1. Crie um Deployment do Nginx no seu cluster:


✅ 2. Crie os quatro Services explicados acima e verifique:

- Os IPs atribuídos para cada tipo de Service.
- O acesso ao Nginx a partir de dentro e fora do cluster (conforme aplicável).

✅ 3. Crie um pod temporário para testar o acesso ao ClusterIP e ExternalName:

✅ 4. Documente as diferenças encontradas e onde cada tipo de Service é mais apropriado.





