# Volumes no Kubernetes
## 🎯 Objetivo
Compreender os diferentes tipos de volumes disponíveis no Kubernetes, suas aplicações práticas, limitações e como utilizá-los para armazenamento efêmero e persistente em aplicações.

## 📘 1. Introdução aos Volumes no Kubernetes
O que são Volumes?
No Kubernetes, um volume é um diretório acessível a partir de um ou mais containers de um pod. Volumes resolvem limitações do sistema de arquivos temporário de containers (que se perde ao reiniciar o pod).

**Características**:
- Sobrevivem à reinicialização de containers (mas não necessariamente de pods).
- Podem ser efêmeros ou persistentes;
- Suportam diversas fontes de dados: memória, diretórios locais, diretórios do host, discos remotos etc.

**Limitações**:
- Volumes associados ao ciclo de vida do pod (exceto os persistentes via PVC/PV).
- Alguns tipos não funcionam em ambientes distribuídos (ex: hostPath).

**Os principais tipos de Volumes**
- **Ephemeral Volumes**: Só existe enquanto o POD existir
- **Host Volumes**: Mapelado algum device, diretório no host local com o container
- **Persistent Volumes (PV)**: Criação de uniades de armazenamento que podem ser local ou remota
- **Volume Class**: Cria (PVs) dinamicamente em ambiente de Cloud de acordo com a demanda o app.

## 📂 2. Ephemeral Volumes
São volumes temporários que existem apenas enquanto o pod estiver em execução.

### 📌 2.1 emptyDir
Cria um diretório vazio que é compartilhado entre os containers do pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
    - name: app
      image: busybox
      command: [ "sh", "-c", "echo Hello > /data/msg; sleep 3600" ]
      volumeMounts:
        - mountPath: /data
          name: temp-storage
  volumes:
    - name: temp-storage
      emptyDir: {}
```      
### 📌 2.2 emptyDir com memória (medium: Memory)
Útil para armazenar dados temporários com performance alta (RAM).

```yaml
volumes:
  - name: temp-storage
    emptyDir:
      medium: Memory
```      
> ⚠️ Atenção: usa RAM do nó — limitado pela memória disponível.

## 📁 3. Volumes com acesso ao sistema de arquivos do Host
### 📌 3.1 hostPath
Permite montar um diretório do host dentro do pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
    - name: app
      image: busybox
      command: [ "sh", "-c", "ls /data; sleep 3600" ]
      volumeMounts:
        - mountPath: /data
          name: host-volume
  volumes:
    - name: host-volume
      hostPath:
        path: /tmp/k8s-data
        type: DirectoryOrCreate
```

### 📌 3.2 subPath
Há situações que o mesmo volume é utilizado por vários containers, mas se deseja montar apenas um subdiretório específico do volume.

Image um diretório no host com a seguinte estrutura:
```yaml
/dados/
    dir01/
       -file01.txt
    dir02/
       -file02.txt
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
    - name: container01
      image: busybox
      command: [ "sh", "-c", "echo 'oi' > file01.txt; sleep 3600" ]
      volumeMounts:
        - name: host-volume
          mountPath: /tmp/
          subPath: dir01

    - name: container02
      image: busybox
      command: [ "sh", "-c", "echo 'oi' > file02.txt; sleep 3600" ]
      volumeMounts:
        - name: host-volume
          mountPath: /tmp/
          subPath: dir02
  volumes:
    - name: host-volume
      hostPath:
        path: /dados
        type: DirectoryOrCreate
```    

## 🗄️ 4. Volumes Persistentes com PersistentVolume e PersistentVolumeClaim

O uso de PersistentVolume (PV) e PersistentVolumeClaim(PVC) são recursos de gerenciamento de volumes no kubernetes que abstrai detalhes de como é consumido e fornecido um armazenamento.

O PersistentVolume, refere-se ao dispositivo físico atrelado a algum tipo de implementação de storage, que pode ser remoto (NFS), local (hostPath, ICSI,...) ou em algum Cloud Provider

Basicamente, temos 3 ações:
1. Cria-se as unidades de armazenamos (PersistentVolumes - PV) no cluster. Sua criação pode ser manual, ou automática em cloud utilizando um StorageClass
2. Requisita-se algum PV com um PersistentVolumeClaim(PVC). Ao requisitar, pode-se consumir apenas parte do PV.
3. Vincula o PVC a um POD, montando seu conteúdo no container.

### 📌 4.1 PersistentVolume com hostPath
Define o volume físico no cluster.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

**🔁 Reclaim Policy**
A política persistentVolumeReclaimPolicy define o comportamento do volume quando um PVC é deletado.

|Política|Descrição|
|--------|---------|
| `Retain`  | Dados são mantidos após exclusão do PVC. O administrador deve limpar manualmente. |
| `Delete`  | O volume e os dados são excluídos automaticamente quando o PVC é deletado. |
| `Recycle` | **Obsoleto**. Apaga os dados e torna o volume disponível novamente (não recomendado). |

### 📌 4.2 PersistentVolumeClaim
Requisição de volume feita por um usuário ou aplicação.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-hostpath
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```      
### 📌 4.3 Uso do PVC em um Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
    - name: container01
      image: busybox
      command: [ "sh", "-c", "echo 'oi' > /app/data/file01.txt; sleep 3600" ]
      volumeMounts:
        - name: my-volume
          mountPath: /app/data
  volumes:
  - name: my-volume
      persistentVolumeClaim:
      claimName: pvc-hostpath
```
