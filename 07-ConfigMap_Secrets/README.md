# 🔍 O que são ConfigMap e Secret no Kubernetes?
No Kubernetes, ConfigMaps e Secrets são recursos que permitem separar a configuração da aplicação (por exemplo, variáveis de ambiente, strings de conexão, parâmetros de inicialização) do código da aplicação em si.

- ConfigMap: Usado para armazenar dados de configuração em 
formato de chave-valor. Ideal para dados que não são 
sensíveis, como parâmetros de configuração.
- Secret: Parecido com o ConfigMap, mas usado para 
armazenar dados sensíveis, como senhas, tokens de API 
e certificados. O Kubernetes faz o gerenciamento desses 
dados de forma mais segura (por exemplo, usando Base64 e, 
em alguns casos, mecanismos de criptografia).

## ⚠️ Limitações
- ✅ Separação da configuração e do código
- ✅ Fácil atualização sem reconstruir a imagem
- ❌ Secrets não são criptografados por padrão
- ❌ ConfigMaps podem expor dados sensíveis se usados para este fim
- ✅ Secrets podem ser montados como volumes ou injetados como variáveis

## Exemplos de uso [Variável ambiente]

**Exemplo 1: Criando e usando um ConfigMap**
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: meu-configmap
data:
  APP_MODE: "production"
  APP_COLOR: "blue"
---
# pod-com-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-com-configmap
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    envFrom:
    - configMapRef:
        name: meu-configmap
```
Aplicando...
```bash
kubectl apply -f configmap-exemplo1.yaml
```

**🔒 Exemplo de uso de Secret**

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: meu-secret
type: Opaque
data:
  SENHA_DB: c2VuaGExMjM=  # senha123 em base64
---
# pod-com-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-com-secret
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo $SENHA_DB && sleep 3600"]
    env:
    - name: SENHA_DB
      valueFrom:
        secretKeyRef:
          name: meu-secret
          key: SENHA_DB
```
Aplicando...
```bash
kubectl apply -f secret-exemplo1.yaml
```

Também seria possível criar esse secret acima desta forma:
```bash
kubectl create secret generic meu-secret --from-literal=SENHA_DB=senha123
```


## Exemplo de uso [Arquivo de Configuração]
É possível também utilizar o `configmap` para armazenar um texto, e esse texto pode ser o conteúdo do arquivo de configuração de uma aplicação.

No exemplo a seguir, vamos alterar o arquivo de configuração do nginx utilizando este recurso.

Suponha que temos um arquivo de configuração nginx.conf:

```nginx
# nginx.conf
server {
  listen 80;
  server_name localhost;
  location / {
    root /usr/share/nginx/html;
    index index.html;
  }
}
```
```bash
kubectl create configmap nginx-config --from-file=nginx.conf
kubectl get configmap nginx-config -o yaml
```

Agora vamos usar este arquivo de configuração no nginx

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
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
          image: nginx:1.25
          volumeMounts:
            - name: nginx-config-volume
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
      volumes:
        - name: nginx-config-volume
          configMap:
            name: nginx-config
```

**Explicação:**

- O ConfigMap nginx-config é montado no contêiner como um arquivo (nginx.conf) diretamente em /etc/nginx/nginx.conf.

- O subPath: nginx.conf garante que apenas o arquivo específico seja montado, não todo o diretório.