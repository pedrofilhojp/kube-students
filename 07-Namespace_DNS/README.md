# Namespaces e DNS no Kubernetes

## 🚀 Introdução: Namespaces no Kubernetes
**Namespaces** são uma forma de organizar recursos em clusters Kubernetes. Eles permitem que múltiplos projetos ou equipes compartilhem o mesmo cluster sem interferirem entre si.
Cada namespace oferece um “espaço de nomes” isolado para objetos como pods, services e configmaps.

Características principais:

- Isolamento lógico de recursos.
- Útil para ambientes de produção, desenvolvimento e testes no mesmo cluster.
- Recursos padrão (como nodes e volumes) não são isolados.

Limitações:

- Não provêm isolamento de segurança (não é como um hypervisor ou VM).
- Namespaces não separam recursos de hardware (CPU/RAM do cluster são compartilhados).

## 🟢 Parte 1: Criando Namespaces e Deployments
Vamos criar dois namespaces: dev e prod.
Em cada namespace, você vai criar um Deployment rodando o Nginx.

**Passos:**
1. Crie os namespaces:

```bash
kubectl create namespace dev
kubectl create namespace prod
```

2. Crie um Deployment Nginx em cada namespace:

```bash
kubectl create deployment nginx-dev --image=nginx --namespace=dev
kubectl create deployment nginx-prod --image=nginx --namespace=prod
```

3. Exponha cada Deployment como um Service do tipo ClusterIP:

```bash
kubectl expose deployment nginx-dev --port=80 --target-port=80 --name=nginx-svc --namespace=dev
kubectl expose deployment nginx-prod --port=80 --target-port=80 --name=nginx-svc --namespace=prod
```
## 🚀 Introdução: DNS no Kubernetes
O DNS interno do Kubernetes permite que os pods se comuniquem usando nomes amigáveis em vez de IPs.
Por padrão, cada namespace tem seu próprio domínio DNS, e o kube-dns/CoreDNS gerencia esses nomes.

Características principais:

- Resolução automática de nomes de Service e Pod.
- Estrutura de nomes: <service>.<namespace>.svc.cluster.local
(ex.: nginx-svc.dev.svc.cluster.local).

Limitações:

- Não resolve nomes externos automaticamente (usa DNS externo para isso).
- Só funciona dentro do cluster.

## 🟢 Parte 2: Testando DNS no Cluster
**Passos:**
1. Crie um Pod de teste no namespace dev para checar a resolução DNS:

```bash
kubectl run test-dns --image=busybox:1.28 --restart=Never -it --namespace=dev -- sh
```
2. Dentro do Pod de teste, verifique o DNS do serviço do Nginx no namespace dev:

```sh
nslookup nginx-svc
```
3. Teste o DNS do serviço Nginx no namespace prod (é necessário o namespace no nome):

```sh
nslookup nginx-svc.prod
```
4. Faça um teste de conexão HTTP ao Nginx no dev:

```sh
wget -qO- nginx-svc
```
## ⚡ Desafio extra
**💡 Crie um arquivo YAML para descrever:**

- Os namespaces dev e prod;
- O Deployment do Nginx;
- O Service para expor o Deployment.

> **Dica**: Use arquivos nginx-deployment.yaml e nginx-service.yaml em conjunto.

## 🎯 Perguntas para reflexão
- O que aconteceria se você tentasse acessar nginx-svc a partir de um Pod no default namespace?
- Qual é a vantagem de usar nomes DNS como nginx-svc.prod em vez de IPs?
- Como você pode expor o serviço do Nginx para fora do cluster?

## 💾 Resultado esperado
Ao final, você deverá:

- Ter os dois namespaces funcionando (dev, prod).
- Confirmar que o Nginx em dev e prod funciona e responde via DNS.
- Entender como o DNS resolve nomes de services em diferentes namespaces