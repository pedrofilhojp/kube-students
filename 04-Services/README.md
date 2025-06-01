# 🚀 Exercício: Explorando os tipos de Services no Kubernetes

Neste exercício, vamos entender os diferentes tipos de Services disponíveis no Kubernetes e como utilizá-los para expor aplicações — usando o **Nginx** como exemplo. Ao final, você será capaz de criar e diferenciar cada tipo de Service.

Para mais informações, utilize a documentação do Kubernetes, [Clique aqui](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/)

---

Antes de iniciar, e de forma bem simples o objetiva, vamos compreender que um **Service** é um recurso de LoadBalance com capacidade de selecionar os PODs de uma aplicação dinamicamente. Ou seja, se uma aplicação que tinha 5 réplicas aumentou para 10, o Service consegue selecinar as novas replicas e balancear o trafego de rede também para elas.

Há vários algorítmos que se pode utilizar em um Loadbalance, mas o Service trabalha apenas com ***Round-Robin***. 

Há apenas 4 tipos de Services no kubernetes:

- **NodePort**: Utilizado para expor as aplicações apenas dentro do cluster, para uma aplicação acessar outra;
- **ClusterIP**: Expor a aplicação para fora do cluster em ambiente **NÃO CLOUD**, privada ou pública;
- **LoadBalance**: O mesmo que ClusterIP, porém para ambientes de Cloud, isso quer dizer que, na AWS por exemplo, você utilizará o LoadBalance para expor a aplicação e você acessá-la;
- **ExternalName**: Mapear algum registro DNS externo para algum service interno no Cluster.

**Porque acessar a aplicação pelo SERVICE e não diretamente o POD?**

Para acessar a aplicação, precisamos do IP do POD, o problema é que, quando o POD cai ele não sobe com o mesmo IP. Além disso, a aplicação pode necessitar de várias réplicas para suportar a carga de acesso dos usuários.

O Service também sobe um endereço IP exclusivo para ele, mas, esse IP não é alterado, logo, se o POD cair e subir novamente com outro IP, o service consegue identificar o IP do novo POD e não terá problemas no acesso.

## 1️⃣ Service ClusterIP

**Uso e limitações:**

O tipo **ClusterIP** é o padrão ao criar um Service no Kubernetes. Ele cria um IP interno acessível apenas dentro do cluster, permitindo a comunicação entre pods e outros recursos internos, mas não expondo o Service externamente.

**Limitações:**
- Não pode ser acessado fora do cluster.
- Ideal para aplicações internas, como backends ou bancos de dados.

**Criação do Service ClusterIP para Nginx:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```
Comando para criar o Deployment do Nginx:

```bash
kubectl create deployment nginx --image=nginx
```

## 2️⃣ NodePort
Uso e limitações:

O tipo NodePort expõe o Service em um porto específico de cada nó do cluster. Assim, o Service pode ser acessado externamente pelo IP de qualquer nó e pelo NodePort atribuído (geralmente na faixa 30000-32767).

Limitações:

Cada Service usa uma porta fixa do nó, que pode causar conflitos se não for bem gerenciado.

Menos eficiente e seguro que LoadBalancer para ambientes de produção.

Criação do Service NodePort para Nginx:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
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
Acesse o Nginx em:

```cpp
http://<IP_DO_NO>:30080
```
## 3️⃣ LoadBalancer
Uso e limitações:

O tipo LoadBalancer cria um balanceador de carga externo (quando disponível no provedor de nuvem) e direciona o tráfego para os pods. É ideal para aplicações expostas publicamente, como sites ou APIs.

Limitações:

Depende do suporte do provedor de nuvem (não funciona em clusters locais sem um load balancer externo configurado).

Pode gerar custos adicionais.

Criação do Service LoadBalancer para Nginx:

yaml
Copiar
Editar
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
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
kubectl get svc nginx-loadbalancer
```

## 4️⃣ ExternalName
Uso e limitações:

O tipo ExternalName permite expor um Service para um nome DNS externo. Ele não cria um proxy nem expõe um IP dentro do cluster. Apenas fornece um alias DNS para um recurso externo.

Limitações:

Não faz roteamento interno de tráfego.

Apenas um mapeamento de nome DNS — não gera pods ou endpoints.

Exemplo de Service ExternalName apontando para um servidor externo:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-nginx
spec:
  type: ExternalName
  externalName: nginx.example.com
```

Neste caso, qualquer requisição para external-nginx.default.svc.cluster.local será redirecionada para nginx.example.com.

## 🚀 Resumo das diferenças

| Tipo de Service   | Acessível de fora do cluster? | Load balancing?      | Usos típicos                 |
|--------------------|------------------------------|----------------------|------------------------------|
| **ClusterIP**      | ❌                            | ✅ (interno)         | Comunicação interna          |
| **NodePort**       | ✅ (pelo IP do nó)           | ✅ (básico)          | Testes, exposições simples   |
| **LoadBalancer**   | ✅ (público, se suportado)   | ✅ (com Load Balancer) | Produção em nuvem            |
| **ExternalName**   | ✅ (via DNS)                 | ❌                    | Alias DNS para recursos externos |


## 📝 Tarefas
✅ 1. Crie um Deployment do Nginx no seu cluster:

```bash
kubectl create deployment nginx --image=nginx
```
✅ 2. Crie os quatro Services explicados acima e verifique:

Os IPs atribuídos para cada tipo de Service.

O acesso ao Nginx a partir de dentro e fora do cluster (conforme aplicável).

✅ 3. Crie um pod temporário para testar o acesso ao ClusterIP e ao ExternalName:

```bash
kubectl run curlpod --image=radial/busyboxplus:curl -it --restart=Never
```
Use:

```bash
curl nginx-clusterip
curl external-nginx
```
✅ 4. Documente as diferenças encontradas e onde cada tipo de Service é mais apropriado.

🎉 Bons estudos! 🚀






