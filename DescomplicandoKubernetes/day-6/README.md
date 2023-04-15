# Descomplicando o Kubernetes - Expert Mode

## DAY-6
&nbsp;

&nbsp;
### Conteúdo do Day-6

- [Descomplicando o Kubernetes - Expert Mode](#descomplicando-o-kubernetes---expert-mode)
  - [DAY-6](#day-6)
    - [Conteúdo do Day-6](#conteúdo-do-day-6)
  - [Inicio da aula do Day-6](#inicio-da-aula-do-day-6)
    - [O que iremos ver hoje?](#o-que-iremos-ver-hoje)
      - [O que são volumes?](#o-que-são-volumes)
        - [Volumes do tipo `emptyDir`](#volumes-do-tipo-emptydir)
        - [Storage Class](#storage-class)
        - [PV - Persistent Volume](#pv---persistent-volume)
        - [PVC - Persistent Volume Claim](#pvc---persistent-volume-claim)
    - [A sua lição de casa](#a-sua-lição-de-casa)
  - [Desafio do Day-6](#desafio-do-day-6)
  - [Final do Day-6](#final-do-day-6)

## Inicio da aula do Day-6

&nbsp;
### O que iremos ver hoje?

Durante o dia de hoje nós vamos aprender tudo sobre volumes, hoje é o dia de você finalmente descomplicar os volumes no Kubernetes! \o/

Hojel nós iremos entender e configurar o que é um `configmap`, um `persistente volume (PV)` e um `persistent volume claim (PVC)`! E para isso iremos utilizar como exemplo diferentes tipos de clusters Kubernetes! Calma, eu explico melhor!

Para ajudar no nosso aprendizado sobre volumes, vamos utlizar diferentes clusters Kubernetes! Vamos ter exemplos utilizando `EKS`, `kind` e instâncias em cloud providers.

Então fique ciente de que hoje é o dia onde você irá descomplicar volumes no Kubernetes! #VAIIII

&nbsp;

#### O que são volumes?

Para simplificar o seu entendimento nesse momento, volumes nada mais são do que um diretório dentro do `Pod` que pode ser utilizado para armazenar dados. Eles podem ser utilizados para armazenar dados que precisam ser persistidos, como por exemplo, dados de um banco de dados, ou dados de um sistema de arquivos distribuído.

Quando estamos falando sobre volumes no Kubernetes, precisamos entender que temos basicamente dois tipos de volumes, os `ephemeral volumes` e os `persistent volumes`.

Os `ephemeral volumes`, que inclusive já vimos durante o treinamento o `emptyDir`, são volumes que são criados e destruídos junto com o `Pod`. Ele é um volume também, porém com uma diferença, ele não é persistente. Caso ocorra algum problema com o `Pod` e ele seja removido, o `emptyDir` também será removido.

Agora quando estamos falando sobre volumes do tipo `persistent volumes`, estamos falando sobre volumes que são criados e não são destruídos junto com o `Pod`, eles são persistidos, são volumes que seus dados são mantidos mesmo que o `Pod` seja removido.

Esse tipo de volume é super importante para aplicações que precisam armazenar dados que precisam ser mantidos mesmo que o `Pod` seja removido, como por exemplo, um banco de dados.

##### Volumes do tipo `emptyDir`

Como já vimos anteriormente, os volumes do tipo `emptyDir` são volumes que são criados e destruídos junto com o `Pod`. Eles são bem úteis para aplicações que precisam armazenar dados temporários, como por exemplo, dados de cache.

A sua definição é bastante simples, e não precisa de criar nada além do `Pod` que irá utilizar o `emptyDir`. Não será necessário criar um `storage class` ou um `persistent volume claim (PVC)`.

Para que possamos criar um `emptyDir` para o nosso exemplo, vamos utilizar uma solução que foi criada junto com o pessoal que me acompanha nas lives na Twitch, vamos utilizar o `Giropops-Senhas`! \o/

Primeira coisa, deixa eu te explicar o que é o `Giropops-Senhas`. O `Giropops-Senhas` é uma app bem simples e que foi criada somente para ser utilizada em exemplos de aulas sobre Kubernetes, observabilidade ou seja lá o que foi interessante. :D

Ela é uma app bem simples que utiliza Python e Flask para criar um gerador de senhas. Essa aplicação utiliza o `Redis` como cache para guardar as últimas senhas geradas.

O que vamos fazer aqui no nosso exemplo é criar um `emptyDir` para o `Redis` e assim, quando o `Pod` for removido, os dados do `Redis` também serão removidos. Bora começar!

ahhh, caso você queira ver o código fonte da aplicação, você pode acessar o repositório do [Giropops-Senhas](https://github.com/badtuxx/giropops-senhas).

Para realizar o deploy da nossa app e do `Redis`, vamos utilizar os seguintes arquivos:

- giropops-senhas-deployment.yaml
- redis-deployment.yaml

Eu ainda não quero entrar em detalhes sobre `Services`, pois ainda não vimos, mas aqui ainda teremos dois outros arquivos, que são os arquivos de `Service` para o `Giropops-Senhas` e para o `Redis`.

Não se preocupe, quando chegarmos na parte de `Services`, eu vou explicar melhor sobre eles.

Bem, vamos aos arquivos!

Primeiro vamos criar o `Deployment` do `Giropops-Senhas`, para isso vamos utilizar o arquivo `giropops-senhas-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: giropops-senhas
  name: giropops-senhas
spec:
  replicas: 3
  selector:
    matchLabels:
      app: giropops-senhas
  template:
    metadata:
      labels:
        app: giropops-senhas
    spec:
      containers:
      - image: linuxtips/giropops-senhas:1.0
        name: giropops-senhas
        ports:
        - containerPort: 5000
        imagePullPolicy: Always
        resources:
          limits:
            memory: "256Mi"
            cpu: "500m"
          requests:
            memory: "128Mi"
            cpu: "250m"
```

&nbsp;

Nele, como você pode ver, não tem nada de novo, somente a definição de um `Deployment` com três réplicas do `Giropops-Senhas`, com alguns limites de recursos e com a imagem do `Giropops-Senhas` que está no Docker Hub.

Agora vamos criar o `Deployment` do `Redis`, para isso vamos utilizar o arquivo `redis-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis
        name: redis
        ports:
          - containerPort: 6379
        resources:
          limits:
            memory: "256Mi"
            cpu: "500m"
          requests:
            memory: "128Mi"
            cpu: "250m"
```

&nbsp;

Mesma coisa, somente a definição de um `Deployment` com uma réplica do `Redis`, com alguns limites de recursos e com a imagem do `Redis` que está no Docker Hub.

Agora vamos criar o `Service` do `Giropops-Senhas`, para isso vamos utilizar o arquivo `giropops-senhas-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: giropops-senhas
  labels:
    app: giropops-senhas
spec:
  selector:
    app: giropops-senhas
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
      name: tcp-app
  type: ClusterIP
```

&nbsp;

Eu não vou entrar em detalhes sobre o `Service` agora, mas você pode ver que ele é bem simples, somente a definição de um `Service` do tipo `ClusterIP` para o `Giropops-Senhas`.

Agora vamos criar o `Service` do `Redis`, para isso vamos utilizar o arquivo `redis-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    app: redis
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
  type: ClusterIP
```

&nbsp;

Pronto! Já temos os nossos arquivos prontos para realizar o deploy da nossa aplicação.

Mas você deve ter notado que não temos ainda a definição do nosso `emptyDir`, certo? Então vamos criar ele agora!

Para isso, vamos utilizar o arquivo `redis-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis
        name: redis
        ports:
          - containerPort: 6379
        resources:
          limits:
            memory: "256Mi"
            cpu: "500m"
          requests:
            memory: "128Mi"
            cpu: "250m"
        volumeMounts:
          - name: redis-data
            mountPath: /data
      volumes:
      - name: redis-data
        emptyDir: {}
```

&nbsp;

Pronto, agora já adicionamos o `emptyDir` no nosso `Deployment` do `Redis`.

A definição é bastante simples, e vou detalhar abaixo:

```yaml
        volumeMounts: # Aqui definimos os volumes que serão montados no container
          - name: redis-data # Aqui definimos o nome do volume
            mountPath: /data # Aqui definimos o caminho onde o volume será montado
    volumes: # Aqui definimos os volumes que serão criados
    - name: redis-data # Aqui definimos o nome do volume
      emptyDir: {} # Aqui definimos o tipo do volume, no caso um emptyDir
```

&nbsp;

Vamos lembrar que a entrada `VolumeMounts` é uma lista de objetos que descrevem os pontos de montagem de volume no container, e ela é definida dentro do container. Já a entrada `Volumes` é uma lista de objetos que descrevem os volumes que podem ser montados nos containers, e ela é definida dentro do `Pod`.

Pronto, agora chegou o momento de realizar o deploy da nossa aplicação!

Para isso, vamos utilizar o comando `kubectl apply -f .`:

```bash
kubectl apply -f giropops-senhas-deployment.yaml
kubectl apply -f giropops-senhas-service.yaml
kubectl apply -f redis-deployment.yaml
kubectl apply -f redis-service.yaml
```

&nbsp;

Pronto! Nosso deploy foi realizado com sucesso!

Vamos verificar se os nossos `Pods` estão rodando?

Para isso, vamos utilizar o comando `kubectl get pods`:

```bash
kubectl get pods
```

&nbsp;

```bash
NAME                               READY   STATUS    RESTARTS   AGE
giropops-senhas-5b5d68858-ffngk    1/1     Running   0          4s
giropops-senhas-5b5d68858-nklgj    1/1     Running   0          4s
giropops-senhas-5b5d68858-vk7n2    1/1     Running   0          4s
redis-deployment-9b9599786-x2xdc   1/1     Running   0          3s
```

&nbsp;

Muito bom! Todos os nossos `Pods` estão rodando!

Vamos fazer o seguinte, vamos acessar o `Giropops-Senhas` e vamos gerar algumas senhas, para isso vamos utilizar o comando `kubectl port-forward`:

```bash
kubectl port-forward service/giropops-senhas 5000:80
```

&nbsp;

Agora acessamos o `Giropops-Senhas` no endereço `http://localhost:5000` em um navegador:

![Giropops-Senhas](giropops-senhas-1.png)

&nbsp;

Clique no botão `Gerar Senha` e vamos gerar 3 senhas:

![Giropops-Senhas](giropops-senhas-2.png)

&nbsp;

Pronto! Agora vamos verificar se as senhas foram geradas no `Redis`:

```bash
kubectl exec -it redis-deployment-9b9599786-x2xdc -- redis-cli
```

&nbsp;

Agora vamos pegar as senhas que foram geradas dentro da cli do `Redis`:

```bash
> keys *
1) "senha"
```

& nbsp;

Agora que já sabemos o nome da nossa chave, vamos pegar os últimos valores da nossa chave. Como nós já criamos três senhas, vamos pegar os últimos três valores:

```bash
> lrange senha -3 -1
```

&nbsp;

```bash
1) "gU'n)D*7"
2) "U/dPl&K*"
3) "1\\yDWJ(*"
```

&nbsp;

Vamos comparar com as senhas que foram geradas no `Giropops-Senhas`:

![Giropops-Senhas](giropops-senhas-3.png)

&nbsp;

Pronto, já estamos entendendo o que está acontecendo, certo? Agora é o momento de testarmos o o n0sso `emptyDir`!

Para isso, vamos remover o `Pod` do `Redis`:

```bash
kubectl delete pod redis-deployment-9b9599786-x2xdc
```

&nbsp;

```bash
NAME                               READY   STATUS    RESTARTS   AGE
giropops-senhas-5b5d68858-ffngk    1/1     Running   0          5m47s
giropops-senhas-5b5d68858-nklgj    1/1     Running   0          5m47s
giropops-senhas-5b5d68858-vk7n2    1/1     Running   0          5m47s
redis-deployment-9b9599786-f2mqq   1/1     Running   0          3s
```

&nbsp;

Como podemos ver, o `ReplicaSet` criou um novo `Pod` para o `Redis`, mas a nossa tarefa é verificar se os dados ainda permanecem no `Redis`!

Vamos conectar mais uma vez na console do `Redis`, e assim verificar se tudo está conforme esperamos.


```bash
kubectl exec -it redis-deployment-9b9599786-f2mqq -- redis-cli
```

&nbsp;

```bash
127.0.0.1:6379> KEYS *
(empty array)
127.0.0.1:6379> 
```

&nbsp;

Como podemos notar, nem a chave `senhas` existe mais, pois desde que matamos o nosso `Pod`, não criamos mais nenhuma senha. Com isso podemos ter a certeza de que os dados que estavam no volume se foram. :)

Simples, não?!?

Mas agora é o seguinte, vamos imaginar que esse dados fossem importantes, e precisamos garantir que os dados não se perderão, caso o `Pod` for removido.

Para isso vamos precisar começar a olhar com carinho as formas de volumes persistentes que temos no Kubernetes. Bora?


&nbsp;


##### Storage Class

Uma StorageClass no Kubernetes é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Essas classes de armazenamento podem ser usadas para provisionar dinamicamente PersistentVolumes (PVs) de acordo com os requisitos dos PersistentVolumeClaims (PVCs) criados pelos usuários.

A StorageClass é útil para gerenciar e organizar diferentes tipos de armazenamento, como armazenamento em disco rápido e caro ou armazenamento em disco mais lento e barato. Além disso, a StorageClass pode ser usada para definir diferentes políticas de retenção, provisionamento e outras características de armazenamento específicas.

Os administradores do cluster podem criar e gerenciar várias StorageClasses para permitir que os usuários finais escolham a classe de armazenamento adequada para suas necessidades.

Cada StorageClass é definida com um provisionador, que é responsável por criar PersistentVolumes dinamicamente conforme necessário. Os provisionadores podem ser internos (fornecidos pelo próprio Kubernetes) ou externos (fornecidos por provedores de armazenamento específicos).

Inclusive os provisionadores podem ser diferentes para cada provedor de nuvem ou onde o Kubernetes está sendo executado. Vou listar alguns provisionadores que são usados e seus respectivos provedores:

- `kubernetes.io/aws-ebs`: AWS Elastic Block Store (EBS)
- `kubernetes.io/azure-disk`: Azure Disk
- `kubernetes.io/gce-pd`: Google Compute Engine (GCE) Persistent Disk
- `kubernetes.io/cinder`: OpenStack Cinder
- `kubernetes.io/vsphere-volume`: vSphere
- `kubernetes.io/no-provisioner`: Volumes locais
- `kubernetes.io/host-path`: Volumes locais

E se você estiver usando o Kubernetes em um ambiente local, como o Minikube, o provisionador padrão é o `kubernetes.io/host-path`, que cria volumes PersistentVolume no diretório do host. Já no Kind, o provisionador padrão é o `rancher.io/local-path`, que cria volumes PersistentVolume no diretório do host.

Para ver a lista completa de provisionadores, consulte a documentação do Kubernetes no link [https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner).

&nbsp;

Para você ver os `Storage Classes` disponíveis no seu cluster, basta executar o seguinte comando:

```bash
kubectl get storageclass
```

&nbsp;

```bash
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  21m
```

&nbsp;

Como você pode ver, no Kind, o provisionador padrão é o `rancher.io/local-path`, que cria volumes PersistentVolume no diretório do host.

Já no EKS, o provisionador padrão é o `kubernetes.io/aws-ebs`, que cria volumes PersistentVolume no EBS da AWS.

```bash
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  6h5m
```

&nbsp;

Vamos ver os detalhes do nosso `Storage Class` padrão:

```bash
kubectl describe storageclass standard
```

&nbsp;

```bash
Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"standard"},"provisioner":"rancher.io/local-path","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

&nbsp;

Uma coisa que podemos ver é que o nosso `Storage Class` está com a opção `IsDefaultClass` como `Yes`, o que significa que ele é o `Storage Class` padrão do nosso cluster, com isso todos os `Persistent Volume Claims` que não tiverem um `Storage Class` definido, irão utilizar esse `Storage Class` como padrão.

&nbsp;

Vamos criar um novo `Storage Class` para o nosso cluster Kubernetes no kind, com o nome "local-storage", e vamos definir o provisionador como "kubernetes.io/host-path", que cria volumes PersistentVolume no diretório do host.

```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: giropops
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

&nbsp;

```bash
kubectl apply -f storageclass.yaml
```

&nbsp;

```bash
storageclass.storage.k8s.io/giropops created
```

&nbsp;

Pronto! Agora nós temos um novo `Storage Class` criado no nosso cluster Kubernetes no kind, com o nome "giropops", e com o provisionador "kubernetes.io/no-provisioner", que cria volumes PersistentVolume no diretório do host.

Para saber mais detalhes sobre o `Storage Class` que criamos, execute o seguinte comando:

```bash
kubectl describe storageclass giropops
```

&nbsp;

```bash
Name:            giropops
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"giropops"},"provisioner":"kubernetes.io/no-provisioner","reclaimPolicy":"Retain","volumeBindingMode":"WaitForFirstConsumer"}

Provisioner:           kubernetes.io/no-provisioner
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Retain
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

&nbsp;

Lembrando que criamos esse `Storage Class` com o provisionador "kubernetes.io/no-provisioner", mas você pode criar um `Storage Class` com qualquer provisionador que você quiser, como o "kubernetes.io/aws-ebs", que cria volumes PersistentVolume no EBS da AWS.

&nbsp;

##### PV - Persistent Volume

O PV é um objeto que representa um recurso de armazenamento físico em um cluster Kubernetes. Ele pode ser um disco rígido em um nó do cluster, um dispositivo de armazenamento em rede (NAS) ou mesmo um serviço de armazenamento em nuvem, como o AWS EBS ou Google Cloud Persistent Disk. 

O PV é utilizado para fornecer armazenamento durável, ou seja, os dados armazenados no PV permanecem disponíveis mesmo quando o container é reiniciado ou movido para outro nó.

No Kubernetes, você pode usar várias soluções de armazenamento como Persistent Volumes (PVs). Essas soluções podem ser divididas em dois tipos: armazenamento local e armazenamento em rede. Vou te dar exemplos de algumas opções populares de cada tipo:

**Armazenamento local:**

- HostPath: É uma maneira simples de usar um diretório do nó do cluster como armazenamento. É útil principalmente para testes e desenvolvimento, pois não é apropriado para ambientes de produção, já que os dados armazenados só estão disponíveis no nó específico.

**Armazenamento em rede:**

- NFS (Network File System): É um sistema de arquivos de rede que permite compartilhar arquivos entre várias máquinas na rede. É uma opção comum para armazenamento compartilhado em um cluster Kubernetes.

- iSCSI (Internet Small Computer System Interface): É um protocolo que permite a conexão de dispositivos de armazenamento de blocos, como SAN (Storage Area Network), por meio de redes IP. Pode ser usado como um PV no Kubernetes.

- Ceph RBD (RADOS Block Device): É uma solução de armazenamento distribuído e altamente escalável que oferece suporte ao armazenamento em bloco, objeto e arquivo. Com o RBD, você pode criar volumes de blocos virtualizados que podem ser montados como PVs no Kubernetes.

- GlusterFS: É um sistema de arquivos distribuído e escalável que permite criar volumes de armazenamento compartilhado em vários nós do cluster. Pode ser usado como um PV no Kubernetes.

- Serviços de armazenamento em nuvem: Fornecedores de nuvem como AWS, Google Cloud e Microsoft Azure oferecem soluções de armazenamento que podem ser integradas ao Kubernetes. Exemplos incluem AWS Elastic Block Store (EBS), Google Cloud Persistent Disk e Azure Disk Storage.

Agora que já sabemos o que é um PV, vamos entender como nós podemos utilizar o `kubectl` para gerenciar os PVs.

Primeira coisa, vamos listar os PVs que temos no nosso cluster:

```bash
kubectl get pv -A
```

```bash
No resources found
```

&nbsp;

Com o comando acima estamos listando todos os PVs que temos no nosso cluster, e como podemos ver, não temos nenhum PV criado, ainda. :)


Vamos resolver isso, bora criar um PV?

Para isso, vamos criar um arquivo chamado `pv.yaml`:

```yaml
apiVersion: v1 # Versão da API do Kubernetes
kind: PersistentVolume # Tipo de objeto que estamos criando, no caso um PersistentVolume
metadata: # Informações sobre o objeto
  name: meu-pv # Nome do nosso PV
  labels:
    storage: local
spec: # Especificações do nosso PV
  capacity: # Capacidade do PV
    storage: 1Gi # 1 Gigabyte de armazenamento
  accessModes: # Modos de acesso ao PV
    - ReadWriteOnce # Modo de acesso ReadWriteOnce, ou seja, o PV pode ser montado como leitura e escrita por um único nó
  persistentVolumeReclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
  hostPath: # Tipo de armazenamento que vamos utilizar, no caso um hostPath
    path: "/mnt/data" # Caminho do hostPath, do nosso nó, onde o PV será criado
  storageClassName: standard # Nome da classe de armazenamento que será utilizada
```

&nbsp;

Antes de criar o PV, eu preciso falar um pouquinho mais sobre o arquivo que criamos, principalmente sobre o que temos de diferente em relação aos outros arquivos que criamos até agora.

- `kind: PersistentVolume`: Aqui estamos definindo o tipo de objeto que estamos criando, no caso um `PersistentVolume`.

Outro ponto importante de mencionar é a seção `spec`, que é onde definimos as especificações do nosso PV.

- `spec.capacity.storage`: Aqui estamos definindo a capacidade do nosso PV, no caso 1 Gigabyte de armazenamento.
- `spec.accessModes`: Aqui estamos definindo os modos de acesso ao PV, no caso o modo `ReadWriteOnce`, que significa que o PV pode ser montado como leitura e escrita por um único nó. Aqui nós temos mais alguns modos de acesso:
  - `ReadOnlyMany`: O PV pode ser montado como somente leitura por vários nós.
  - `ReadWriteMany`: O PV pode ser montado como leitura e escrita por vários nós.
- `spec.persistentVolumeReclaimPolicy`: Aqui estamos definindo a política de reivindicação do PV, no caso a política `Retain`, que significa que o PV não será excluído quando o PVC for excluído. Aqui nós temos mais algumas políticas:
  - `Recycle`: O PV será excluído quando o PVC for excluído, mas antes disso ele será limpo, ou seja, todos os dados serão apagados.
  - `Delete`: O PV será excluído quando o PVC for excluído.

Outra seção importante é a seção `hostPath`, que é onde definimos o tipo de armazenamento que vamos utilizar, no caso um `hostPath`. Vou detalhar abaixo os tipos de armazenamento que podemos utilizar:

- `hostPath`: É uma maneira simples de usar um diretório do nó do cluster como armazenamento. É útil principalmente para testes e desenvolvimento, pois não é apropriado para ambientes de produção, já que os dados armazenados só estão disponíveis no nó específico. Ele é ideal em cenários de testes com somente um node.
- `nfs`: É um sistema de arquivos de rede que permite compartilhar arquivos entre várias máquinas na rede. É uma opção comum para armazenamento compartilhado em um cluster Kubernetes.
- `iscsi`: É um protocolo que permite a conexão de dispositivos de armazenamento de blocos, como SAN (Storage Area Network), por meio de redes IP.
- `csi`: Que significa Container Storage Interface, é um recurso que permite a integração de soluções de armazenamento de terceiros com o Kubernetes. O CSI permite que os provedores de armazenamento implementem seus próprios plugins de armazenamento e os integrem ao Kubernetes. É graças ao CSI que podemos utilizar soluções de armazenamento de terceiros, como o AWS EBS, Google Cloud Persistent Disk e Azure Disk Storage.
- `cephfs`: É um sistema de arquivos distribuído e escalável que permite criar volumes de armazenamento compartilhado em vários nós do cluster.
- `local`: É um tipo de armazenamento que permite a criação de volumes locais, onde você pode especificar o caminho do diretório onde os dados serão armazenados. É útil principalmente para testes e desenvolvimento, já que não é apropriado para ambientes de produção, já que os dados armazenados só estão disponíveis no node específico. A diferença entre o `hostPath` e o `local` é que o `local` é um recurso nativo do Kubernetes, enquanto o `hostPath` é um recurso do Kubernetes que utiliza o recurso nativo do Docker e não é recomendado quando estamos com mais de um node no cluster.
- `fc`: É um protocolo que permite a conexão de dispositivos de armazenamento de blocos utilizando redes de fibra óptica. É uma opção comum para armazenamento compartilhado em um cluster Kubernetes.

Eu listei somente os tipos de armazenamento mais comuns, mas você pode encontrar mais informações sobre os tipos de armazenamento no [Kubernetes Docs](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes).

E por último, temos a seção `storageClassName`, que é onde definimos o nome da classe de armazenamento que iremos adicionar o PV.

&nbsp;

Conforme vamos avançando no treinamento, vamos conhecendo mais detalhes sobre cada tipo de armazenamento.

Pronto, tudo está pronto para criarmos o PV.

```bash
kubectl apply -f pv.yaml
persistentvolume/meu-pv created
```

&nbsp;

Vamos listar o nosso PV para ver se ele foi criado corretamente.

```bash
kubectl get pv
```

&nbsp;

```bash
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
meu-pv   1Gi        RWO            Retain           Available           standard                10s
```

&nbsp;

PV criado com sucesso.

Podemos ver que o nosso PV está com o status `Available`, o que significa que ele está disponível para ser utilizado por um PVC.

Vamos ver os detalhes do nosso PV.

```bash
kubectl describe pv meu-pv
```

&nbsp;

```bash
Name:            meu-pv
Labels:          storage=local
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Available
Claim:           
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /mnt/data
    HostPathType:  
Events:            <none>
```

&nbsp;

Dessa forma estamos criando o PV utilizando o provisionador `hostPath`, que é um provisionador para ser utilizado em testes e desenvolvimento, já que os dados armazenados só estão disponíveis no node específico, por isso bora para mais um exemplo, mas agora utilizando o provisionador `nfs`, que é um sistema de arquivos de rede que permite compartilhar arquivos entre várias máquinas na rede.

&nbsp;

Primeira coisa que vamos fazer é criar o diretório que será compartilhado entre os nodes do cluster. Lembrando que para esse exemplo, estou utilizando uma máquina Linux para criar o compartilhamento NFS, mas você pode utilizar qualquer outro sistema operacional, desde que ele tenha suporte ao NFS.

```bash
mkdir /mnt/nfs
```

&nbsp;


Precisamos instalar os pacotes `nfs-kernel-server` e `nfs-common` para que o servidor NFS e o cliente NFS sejam instalados.

```bash
sudo apt-get install nfs-kernel-server nfs-common
```

&nbsp;

Vamos editar o arquivo `/etc/exports`, que é o arquivo de configuração do NFS, e adicionar o diretório que será compartilhado entre os nodes do cluster.

```bash
sudo vi /etc/exports
```

&nbsp;

```bash
/mnt/nfs *(rw,sync,no_root_squash,no_subtree_check)
```

&nbsp;

Onde:

- `/mnt/nfs`: é o diretório que você deseja compartilhar.

- `*`: permite que qualquer host acesse o diretório compartilhado. Para maior segurança, você pode substituir * por um intervalo de IPs ou por IPs específicos dos clientes que terão acesso ao diretório compartilhado. Por exemplo, 192.168.1.0/24 permitiria que todos os hosts na sub-rede 192.168.1.0/24 acessassem o diretório compartilhado.

- `rw`: concede permissões de leitura e gravação aos clientes.

- `sync`: garante que as solicitações de gravação sejam confirmadas somente quando as alterações tiverem sido realmente gravadas no disco.

- `no_root_squash`: permite que o usuário root em um cliente NFS acesse os arquivos como root. Caso contrário, o acesso seria limitado a um usuário não privilegiado.

- `no_subtree_check`: desativa a verificação de subárvore, o que pode melhorar a confiabilidade em alguns casos. A verificação de subárvore normalmente verifica se um arquivo faz parte do diretório exportado.


Agora vamos falar para o NFS que o diretório `/mnt/nfs` está disponível para ser compartilhado.

```bash
sudo exportfs -arv
```

&nbsp;

Maravilha! Vamos agora verificar se o NFS está funcionando corretamente.

```bash
showmount -e
```

&nbsp;

```bash
Export list for localhost:
/mnt/nfs *
```

&nbsp;

Já era! O nosso NFS está funcionando corretamente. \o/

Agora que já temos o nosso NFS funcionando, vamos criar o nosso StorageClass para o provisionador `nfs`.

Para esse exemplo, vamos criar um arquivo chamado `storageclass-nfs.yaml` e adicionar o seguinte conteúdo.

```yaml
apiVersion: storage.k8s.io/v1 # Versão da API do Kubernetes
kind: StorageClass # Tipo de objeto que estamos criando, no caso um StorageClass
metadata: # Informações sobre o objeto
  name: nfs # Nome do nosso StorageClass
provisioner: kubernetes.io/no-provisioner # Provisionador que será utilizado para criar o PV
reclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
volumeBindingMode: WaitForFirstConsumer
parameters: # Parâmetros que serão utilizados pelo provisionador
  archiveOnDelete: "false" # Parâmetro que indica se os dados do PV devem ser arquivados quando o PV for excluído
```

&nbsp;

O Kubernetes não possui um provisionador `nfs` nativo, então não é possível fazer com que o provisionador `kubernetes.io/no-provisioner` crie um PV utilizando um servidor NFS automaticamente, para que isso seja possível, precisamos utilizar um provisionador `nfs` externo, mas isso não é o foco nesse momento, então vamos criar o nosso PV manualmente, afinal de contas, já estamos experts em PVs, certo?

Bora lá!

Então já podemos criar o PV e associa-lo ao Storage Class, e para isso vamos criar um novo arquivo chamado `pv-nfs.yaml` e adicionar o seguinte conteúdo.

```yaml
apiVersion: v1 # Versão da API do Kubernetes
kind: PersistentVolume # Tipo de objeto que estamos criando, no caso um PersistentVolume
metadata: # Informações sobre o objeto
  name: meu-pv-nfs # Nome do nosso PV
  labels:
    storage: nfs # Label que será utilizada para identificar o PV
spec: # Especificações do nosso PV
  capacity: # Capacidade do PV
    storage: 1Gi # 1 Gigabyte de armazenamento
  accessModes: # Modos de acesso ao PV
    - ReadWriteOnce # Modo de acesso ReadWriteOnce, ou seja, o PV pode ser montado como leitura e escrita por um único nó
  persistentVolumeReclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
  nfs: # Tipo de armazenamento que vamos utilizar, no caso o NFS
    server: IP_DO_SERVIDOR_NFS # Endereço do servidor NFS
    path: "/mnt/nfs" # Compartilhamento do servidor NFS
  storageClassName: nfs # Nome da classe de armazenamento que será utilizada
```

&nbsp;

Agora vamos criar o nosso PV.

```bash
kubectl apply -f pv-nfs.yaml
```

&nbsp;

```bash
persistentvolume/meu-pv created
```

&nbsp;


Tudo certo com o nosso PV, agora eu acho que já podemos passar para o próximo tópico, que é o PVC.

&nbsp;


##### PVC - Persistent Volume Claim

O PVC é uma solicitação de armazenamento feita pelos usuários ou aplicativos no cluster Kubernetes. Ele permite que os usuários solicitem um volume específico, com base em tamanho, tipo de armazenamento e outras características. O PVC age como uma "assinatura" que reivindica um PV para ser usado por um contêiner. O Kubernetes tenta associar automaticamente um PVC a um PV compatível, garantindo que o armazenamento seja alocado corretamente.


Através do PVC, as pessoas podem abstrair os detalhes de cada tipo de armazenamento, permitindo maior flexibilidade e portabilidade entre diferentes ambientes e provedores de infraestrutura. Ele também permite que os usuários solicitem volumes com diferentes características, como tamanho, tipo de armazenamento e modo de acesso.

Todo PVC é associado a um `Storage Class` ou a um `Persistent Volume`. O `Storage Class` é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Já o `Persistent Volume` é um recurso que representa um volume de armazenamento disponível para ser usado pelo cluster.

Vamos criar o nosso primeiro PVC para o PV que criamos anteriormente.

Para isso, vamos criar um arquivo chamado `pvc.yaml` e adicionar o seguinte conteúdo:

```yaml
apiVersion: v1 # versão da API do Kubernetes
kind: PersistentVolumeClaim # tipo de recurso, no caso, um PersistentVolumeClaim
metadata: # metadados do recurso
  name: meu-pvc # nome do PVC
spec: # especificação do PVC
  accessModes: # modo de acesso ao volume
    - ReadWriteOnce # modo de acesso RWO, ou seja, somente leitura e escrita por um nó
  resources: # recursos do PVC
    requests: # solicitação de recursos
      storage: 1Gi # tamanho do volume que ele vai solicitar
  storageClassName: nfs # nome da classe de armazenamento que será utilizada
  selector: # seletor de labels
    matchLabels: # labels que serão utilizadas para selecionar o PV
      storage: nfs # label que será utilizada para selecionar o PV
```

&nbsp;

Aqui nós estamos definindo o nosso PVC, e vou falar um pouco sobre as principais seções do nosso arquivo.

A seção `accessModes` é onde definimos o modo de acesso ao volume, que pode ser `ReadWriteOnce` (RWO), `ReadOnlyMany` (ROM) ou `ReadWriteMany` (RWM). O RWO significa que o volume pode ser montado como somente leitura e escrita por um nó. O ROM significa que o volume pode ser montado como somente leitura por vários nós. O RWM significa que o volume pode ser montado como leitura e escrita por vários nós.

A seção `resources` é onde definimos os recursos que o PVC irá solicitar. Nesse caso, estamos solicitando um volume de 1Gi.

Ainda temos a seção `storageClassName`, que é onde definimos o nome da classe de armazenamento que iremos associar ao PVC.

E por fim, temos a seção `selector`, que é onde definimos o seletor de labels que será utilizado para selecionar o PV que será associado ao PVC.


Vamos criar o nosso PVC.

```bash
kubectl apply -f pvc.yaml
persistentvolumeclaim/meu-pvc created
```

&nbsp;

Pronto, PVC criado! Vamos conferir se ele foi criado corretamente.

```bash
kubectl get pvc
```

&nbsp;

```bash
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
meu-pvc   Pending                                      nfs            5s
```

&nbsp;

Está lá! Porém o status dele está como `Pending`, vamos ver se tem alguma informação que nos ajude a entender o que está acontecendo.

```bash
kubectl describe pvc meu-pvc
```

&nbsp;

```bash
Name:          meu-pvc
Namespace:     default
StorageClass:  nfs
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age                 From                         Message
  ----    ------                ----                ----                         -------
  Normal  WaitForFirstConsumer  15s (x4 over 1m5s)  persistentvolume-controller  waiting for first consumer to be created before binding
```

&nbsp;

Repare na parte dos eventos, lá diz que o PVC está esperando o primeiro consumidor ser criado antes de ser vinculado. O que isso significa?

Significa que o PVC está esperando que um Pod seja criado para que ele possa ser vinculado ao PV, então bora criar o nosso Pod.

Vamos usar o nosso já conhecido Nginx como exemplo, então vamos criar um arquivo chamado `pod.yaml` e adicionar o seguinte conteúdo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: meu-pvc
      mountPath: /usr/share/nginx/html
  volumes:
  - name: meu-pvc
    persistentVolumeClaim:
      claimName: meu-pvc
```

&nbsp;

O que estamos fazendo aqui é basicamente o seguinte:

* Criando um Pod com o nome `nginx-pod`;
* Utilizando a imagem `nginx:latest` como base;
* Expondo a porta 80;
* Definindo um volume chamado `meu-pvc` e montando ele no caminho `/usr/share/nginx/html` dentro do container;
* Por fim, definindo que o volume `meu-pvc` é um `PersistentVolumeClaim` e que o nome do PVC é `meu-pvc`.

Esse trecho do arquivo `pod.yaml` é responsável por montar o volume `meu-pvc` no caminho `/usr/share/nginx/html` dentro do container.

```yaml
    volumeMounts: # montando o volume no container
    - name: meu-pvc  # nome do volume
      mountPath: /usr/share/nginx/html # caminho onde o volume será montado no container
  volumes: # definindo o volume que será utilizado pelo Pod
  - name: meu-pvc # nome do volume
    persistentVolumeClaim: # tipo de volume, no caso, um PersistentVolumeClaim
      claimName: meu-pvc # nome do PVC
```

Esclarecido! Vamos criar o nosso Pod.

```bash
kubectl apply -f pod.yaml
pod/nginx-pod created
```

&nbsp;

Bora ver se está tudo certo com o nosso Pod.

```bash
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          21s
```

&nbsp;

Parece que sim! Agora vamos ver se o nosso PVC foi vinculado ao PV.

```bash
kubectl get pvc
```

&nbsp;

```bash
NAME      STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
meu-pvc   Bound    meu-pv-nfs   1Gi        RWO            nfs            3m8s
```

&nbsp;

Opa! Temos um vinculo! 

Vamos ver se tem alguma coisa nova na saída do `get pv`.

```bash
kubectl get pv
```

&nbsp;

```bash
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   REASON   AGE
meu-pv-nfs   1Gi        RWO            Retain           Bound    default/meu-pvc   nfs                     3m42s
```

&nbsp;

Agora sim! Temos um PV com o status `Bound` e um PVC com o status `Bound` também. \o/

E pra finalizar o nosso primeiro teste, vamos ver se o nosso Pod está utilizando o nosso volume.

```bash
kubectl describe pod nginx-pod
```

&nbsp;

```bash
Name:             nginx-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-linuxtips-worker/172.18.0.4
Start Time:       Tue, 11 Apr 2023 01:47:48 +0200
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.2.3
IPs:
  IP:  10.244.2.3
Containers:
  nginx:
    Container ID:   containerd://b5946958f63c392c8a77b06811f7859113a1dd260ebcc2113579af6b32c4f549
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:2ab30d6ac53580a6db8b657abf0f68d75360ff5cc1670a85acb5bd85ba1b19c0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 11 Apr 2023 01:47:50 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from meu-pvc (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8874f (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  meu-pvc:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  meu-pvc
    ReadOnly:   false
  kube-api-access-8874f:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  8s    default-scheduler  Successfully assigned default/nginx-pod to kind-linuxtips-worker
  Normal  Pulling    7s    kubelet            Pulling image "nginx:latest"
  Normal  Pulled     6s    kubelet            Successfully pulled image "nginx:latest" in 799.112685ms (799.119928ms including waiting)
  Normal  Created    6s    kubelet            Created container nginx
  Normal  Started    6s    kubelet            Started container nginx
```

&nbsp;

Pronto! O nosso Pod está utilizando o nosso volume! Todo o conteúdo que for criado dentro do Pod será armazenado no nosso volume, e mesmo que o Pod seja removido, o conteúdo não será perdido.

Agora vamos testar o nosso volume. Vamos criar um arquivo HTML simples no diretório `/mnt/data` do nosso servidor NFS.

```bash
echo "<h1>GIROPOPS STRIGUS GIRUS</h1>" > /mnt/data/index.html
```

&nbsp;

Agora vamos ver se o nosso arquivo foi criado.

```bash
kubectl exec -it nginx-pod -- ls /usr/share/nginx/html
```

&nbsp;

```bash
index.html
```

&nbsp;

Está lá! Vamos dar um `curl` de dentro do Pod para ver se o Ngix está servindo o nosso arquivo.

```bash
kubectl exec -it nginx-pod -- curl localhost
```

&nbsp;

```bash
<h1>GIROPOPS STRIGUS GIRUS</h1>
```

Tudo rolando maravilhosamente bem! :D


&nbsp;


### A sua lição de casa

A sua lição de casa é 

&nbsp;
## Desafio do Day-6

Não esqueça de

&nbsp;

## Final do Day-6

Durante o Day-6 você aprendeu 


&nbsp;
