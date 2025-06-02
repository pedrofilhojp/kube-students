# 🌟 Ingress
## O que é?
O Ingress é um recurso (objeto) do Kubernetes que define regras de roteamento HTTP/HTTPS para expor serviços internos do cluster para o mundo externo. Ele atua como uma espécie de "roteador de tráfego" de aplicações.

## Principais características:

- Especifica hosts (por exemplo, app1.example.com) e caminhos (/, /api, etc.).
- Permite configurar TLS (HTTPS).
- É declarativo: você descreve como quer que o tráfego seja roteado, e o IngressController implementa essas regras.
- Não faz nada sozinho! Depende de um IngressController que entenda e aplique essas regras.

## Principais partes de um Ingress:

- rules: definem como rotear tráfego para serviços internos.
- tls: define certificados TLS para HTTPS.
- defaultBackend (opcional): backend padrão para requisições que não casam com nenhuma regra.

**Exemplo de Ingress**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80

```

# IngressController
## O que é?
O IngressController é o componente que escuta e processa recursos Ingress no cluster e cria rotas reais para o tráfego. Ele é responsável por implementar as regras que você descreveu nos Ingress.

## Características principais:

- Funciona como um reverse proxy / balanceador de carga de camada 7 (L7: HTTP/HTTPS).
- Cada IngressController tem seu conjunto de regras, que podem ser filtradas por IngressClass.
- Implementações populares:

    - nginx (Ingress-Nginx Controller)
    - HAProxy
    - Traefik
    - Contoladores de nuvem (ex: AWS ALB Ingress Controller)

## Como funciona?
- Você cria um recurso **Ingress** com as regras de roteamento.
- O IngressController “vigia” esses recursos.
- Ele converte as regras em configurações reais (nginx.conf, haproxy.cfg, etc.).
- Ele gerencia a TLS termination (HTTPS) e balanceamento.

## Exemplo:
Quando você instala o nginx-ingress-controller via Helm ou manifest YAML, ele cria pods que monitoram todos os Ingress e aplicam as regras via nginx.

# IngressClass
## O que é?
O IngressClass é um recurso que define qual IngressController vai gerenciar determinado Ingress. Ele permite que múltiplos IngressControllers coexistam no cluster, cada um responsável por um conjunto de Ingresses.

## Características principais:

- Possui um controller: identificador do controlador que deve gerenciar Ingresses dessa classe.
- É referenciado no Ingress via .spec.ingressClassName.
- Facilita ambientes híbridos onde diferentes tipos de tráfego ou controladores são usados.

## Por que usar IngressClass?
Imagine que você quer:

- Um nginx IngressController para aplicações internas.
- Um Traefik IngressController para aplicações expostas publicamente.

Usando IngressClass, você isola qual IngressController gerencia cada Ingress.

**Configuração do IngressClass**

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
```

**Configuração do Ingress**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example
spec:
  ingressClassName: nginx
  rules:
  - host: app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80

```