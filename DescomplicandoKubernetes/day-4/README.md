# Descomplicando o Kubernetes - Expert Mode

## DAY-4
&nbsp;

&nbsp;
## Conteúdo do Day-4

&nbsp;

## Inicio da aula do Day-4

### O que iremos ver hoje?

Hoje é dia de falar sobre dois objetos muito importantes no Kubernetes, os `ReplicaSets` e os `DaemonSets`.

Nós já sabemos o que é um `Deployment` e também já sabemos o que é um `Pod` no detalhe, então agora vamos conhecer essas duas figuras que estão super conectadas com o `Deployment` e com o `Pod`.
Quando falamos sobre `Deployment` é impossível não falar sobre `ReplicaSet`, pois o `Deployment` é um objeto que cria um `ReplicaSet` e o `ReplicaSet` é um objeto que cria um `Pod`, veja que tudo está conectado.

Já o nosso querido `DaemonSet` é um objeto que cria um `Pod` e esse `Pod` é um objeto que fica rodando em todos os nodes do cluster, super importante para nós, pois é com `DaemonSet` que nós conseguimos garantir que teremos pelo menos um `Pod` rodando em cada node do cluster. Por exemplo, imagine que você precisa de instalar os agente do `Datadog` ou ainda um `exporter` do `Prometheus` em todos os nodes do cluster, para isso você precisa de um `DaemonSet`.


### ReplicaSet

Uma coisa é super importante de saber, quando estamos criando um `Deployment` no Kubernetes, automaticamente estamos criando além do `Deployment` um `ReplicaSet` e esse `ReplicaSet` é quem vai criar os `Pods` que estão dentro do `Deployment`.

Confuso, não?

Não, não é e vou te explicar.

Quando criamos um `Deployment`, o Kubernetes cria um `ReplicaSet` para criar e fazer o gerenciamento das réplicas dos `Pods` em nosso cluster. Ele é o responsável por ficar observando os `Pods` e garantir o número de réplicas que nós definimos no `Deployment`.

É possível criar um `ReplicaSet` sem um `Deployment`, mas não é uma boa prática, pois o `ReplicaSet` não tem a capacidade de fazer o gerenciamento de versões dos `Pods` e também não tem a capacidade de fazer o gerenciamento de `RollingUpdate` dos `Pods`.

E olha que interessante, quando estamos fazendo a atualização de uma versão de um `Pod` com o `Deployment`, o `Deployment` cria um novo `ReplicaSet` para fazer o gerenciamento das réplicas dos `Pods` e quando a atualização termina, o `Deployment` remove as réplicas do `ReplicaSet` antigo e deixa apenas as réplicas do `ReplicaSet` novo.

Mas ele não remove o `ReplicaSet` antigo, ele deixa ele lá, pois ele pode ser usado para fazer um `Rollback` da versão do `Pod` caso algo dê errado. Sim, quando precisamos fazer o `Rollback` de uma atualização em nossos `Pods`, o `Deployment` somente muda o `ReplicaSet` que está sendo usado para fazer o gerenciamento das réplicas dos `Pods`, passando a utilizar o `ReplicaSet` antigo.

Vamos fazer isso na prática?

Acho que vai te ajudar a entender melhor.

### O Deployment e o ReplicaSet

Vamos criar um `Deployment` com o nome de `nginx-deployment` e vamos criar 3 réplicas do `Pod` do `nginx`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
```

&nbsp;

Vamos visualizar o `Deployment` foi criado.

```bash
kubectl get deployments
```

&nbsp;

A nossa saída será parecida com essa.

```bash
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           7s
```

&nbsp;

Simples, eu nós já sabiamos! Jeferson, eu quero saber sobre o `ReplicaSet`!

Calma, pois o nosso querido `Deployment` já criou o `ReplicaSet` para nós.

Vamos visualizar o `ReplicaSet` que foi criado.

```bash
kubectl get replicasets
```

A nossa saída será parecida com essa.

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   1         1         1       37s
```

&nbsp;

Um coisa importante de observar na saída acima é que o `ReplicaSet` tem o mesmo nome do `Deployment` seguido de um sufixo aleatório, e ainda nessa saída podemos saber que o `ReplicaSet` atualmente tem 1 réplica do `Pod` do `nginx` rodando, de acordo com o que nós definimos no `Deployment`.

Vamos aumentar o número de réplicas do `Pod` do `nginx` para 3.

```bash
kubectl scale deployment nginx-deployment --replicas=3
```

&nbsp;

Essa é uma forma de aumentar o número de réplicas do `Pod` do `nginx` sem precisar editar o `Deployment`, eu não recomendo, eu prefiro editar o `Deployment` e fazer o `apply` novamente, mas isso é uma questão de gosto e organização. Eu não gosto da ideia de ter que ficar fazendo `scale` no `Deployment` para aumentar ou diminuir o número de réplicas do `Pod` do `nginx`, sem ter isso registrado no `git`, por exemplo.

Alterando o `Deployment` para 3 réplicas.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
```

&nbsp;

Pronto, você conhece as duas opções de aumentar o número de réplicas do `Pod` do `nginx`, fique a vonta para escolher a que você achar melhor.

Eu vou continuar usando a opção de editar o `Deployment` e fazer o `apply` novamente.

```bash
kubectl apply -f nginx-deployment.yaml
```

&nbsp;

Vamos verificar o nosso `ReplicaSet` novamente.

```bash
kubectl get replicasets
```

&nbsp;

A nossa saída será parecida com essa.

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   3         3         3       5m24s
```

&nbsp;

Perceba que o nome do `ReplicaSet` continua o mesmo, mas o número de réplicas mudou para 3. Quando somente alteramos o número de réplicas do nosso `Deployment`, o `ReplicaSet` permanece o mesmo, afinal sua principal função é fazer o gerenciamento das réplicas do `Pod` do `nginx`.

Agora vamos mudar a versão do `nginx` para a versão 1.19.2.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
```

&nbsp;

Vamos aplicar as alterações.

```bash
kubectl apply -f nginx-deployment.yaml
```

&nbsp;

Pronto, agora o `Deployment` está usando a versão 1.19.2 do `nginx`.

Vamos verificar o nosso `ReplicaSet` novamente.

```bash
kubectl get replicasets
```

&nbsp;

Teremos agora a seguinte saída.

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   0         0         0       8m53s
nginx-deployment-7858bcf56f   1         1         1       13s
```

&nbsp;

Agora sim nós temos um novo `ReplicaSet` com o nome de `nginx-deployment-7858bcf56f`, e o `ReplicaSet` antigo com o nome de `nginx-deployment-6dd8d7cfbd` foi esvaziado, pois ele não faz mais parte do `Deployment`, mas permanece no cluster, pois ele pode ser usado para fazer o rollback para a versão anterior do `nginx`.

Vamos ver um detalhe interessante em nosso `Deployment`.

```bash
kubectl describe deployment nginx-deployment
```

&nbsp;

Olha a linha referente ao `ReplicaSet` que está sendo gerenciado pelo `Deployment`.

```bash
NewReplicaSet:   nginx-deployment-7858bcf56f (3/3 replicas created)
```

&nbsp;

Sim, na saída do `describe` nós podemos ver que o `Deployment` está gerenciando o `ReplicaSet` com o nome de `nginx-deployment-7858bcf56f` e que ele tem 3 réplicas do `Pod` do `nginx` rodando.

Se você quiser fazer o rollback para a versão anterior do `nginx`, basta fazer o seguinte, conforme nós já vimos anteriormente.

```bash
kubectl rollout undo deployment nginx-deployment
```

&nbsp;

Com isso será feito o rollback para a versão anterior do `nginx` e o `ReplicaSet` com o nome de `nginx-deployment-7858bcf56f` será esvaziado e o `ReplicaSet` com o nome de `nginx-deployment-6dd8d7cfbd` será preenchido novamente com 3 réplicas do `Pod` do `nginx`.

Vamos listar novamente os nossos `ReplicaSet`.

```bash
kubectl get replicasets
```

&nbsp;

O que temos agora é isso.

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   3         3         3       15m
nginx-deployment-7858bcf56f   0         0         0       6m28s
```

&nbsp;

E se olharmos o `Deployment` novamente.

```bash
kubectl describe deployment nginx-deployment
```

&nbsp;

Teremos a seguinte saída.

```bash
NewReplicaSet:   nginx-deployment-6dd8d7cfbd (3/3 replicas created)
```

&nbsp;

Simples demais, não é mesmo?

Então agora você já sabe como fazer o gerenciamento de réplicas do `Pod` do `nginx` usando o `Deployment`, e por consequência o `ReplicaSet`.

&nbsp;

#### Criando um ReplicaSet

Como foi falado anteriormente, é possível você criar um `ReplicaSet` sem ser através de um `Deployment`, apesar de insistir, não faça isso, pois o `Deployment` é a forma mais fácil de gerenciar os `ReplicaSet` e a saúde dos `Pods`.

Mas vamos lá, caso você queira criar um `ReplicaSet` sem ser através de um `Deployment`, basta fazer o seguinte.

Para o nosso exemplo, vamos criar um arquivo com o nome de `nginx-replicaset.yaml` e vamos colocar o seguinte conteúdo.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: nginx-app
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
```

&nbsp;

Antes de aplicar o nosso manifesto, vamos entender o que estamos fazendo.
Se você observar o nosso arquivo, não tem nada de novo, ou seja, nada que você já não tenha aprendido até agora. A principal diferença é que agora nós estamos usando o `kind: ReplicaSet` e não o `kind: Deployment`, até o `APIVersion` é a mesmo.

Agora vamos aplicar o nosso manifesto.

```bash
kubectl apply -f nginx-replicaset.yaml
```

&nbsp;

A saída será essa.

```bash
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   3         3         3       21m
nginx-deployment-7858bcf56f   0         0         0       12m
nginx-replicaset              3         3         3       6s
```

&nbsp;

Agora nós temos 3 `ReplicaSet` sendo gerenciados pelo `Kubernetes`, sendo que 2 deles são gerenciados pelo `Deployment` e o outro foi o que acabamos de criar.

Vamos listar os `Pods` que estão rodando.

```bash
kubectl get pods
```

&nbsp;

Tudo rolando magicamente, certo?

Agora vamos fazer um teste, vamos alterar a versão do `nginx` para a versão 1.19.3, para isso vamos editar o nosso arquivo `nginx-replicaset.yaml` e vamos alterar a versão do `nginx` para a versão 1.19.3.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: nginx-app
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - image: nginx:1.19.3
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
```

&nbsp;

Pronto, agora vamos aplicar as alterações.

```bash
kubectl apply -f nginx-replicaset.yaml
```

&nbsp;

Vamos listar os `Pods` novamente.

```bash
kubectl get pods
```

&nbsp;

Perceba que nada aconteceu, ou seja, o `ReplicaSet` não fez o rollout da nova versão do `nginx`, isso acontece porque o `ReplicaSet` não faz o gerenciamento de versões, ele apenas garante que o número de réplicas do `Pod` esteja sempre ativo.

Se você olhar os detalhes do `ReplicaSet` você vai ver que ele está gerenciando 3 réplicas do `Pod` e que a imagem do `nginx` é a versão 1.19.3, porém ele não recriou os `Pods` com a nova versão do `nginx`, ele somente irá fazer isso se você apagar os `Pods` manualmente ou se o `Pod` morrer por algum motivo.

```bash
kubectl describe replicaset nginx-replicaset
```

&nbsp;

Agora vamos apagar um dos `Pods` para que o `ReplicaSet` crie um novo `Pod` com a nova versão do `nginx`.

```bash
kubectl delete pod nginx-replicaset-8r6md
```

&nbsp;

Agora o `ReplicaSet` vai criar um novo `Pod` com a nova versão do `nginx`, e gerando um problema para nós, pois agora nós temos duas versões do `nginx` rodando no nosso cluster.

```bash
kubectl get pods -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{"\t"}{range .spec.containers[*]}{.image}{"\t"}{end}{end}'
```

&nbsp;

Essa é uma forma de você listar os `Pods` e as imagens que estão sendo usadas por eles, eu sei que é bastante esquisito, mas vou explicar o que está acontecendo.

- `kubectl get pods`: esse comando lista todos os Pods no cluster.

- `-o=jsonpath`: esse parâmetro especifica que queremos usar a saída em formato JSONPath para exibir as informações dos Pods.

- `'{range .items[*]}{"\n"}{.metadata.name}{"\t"}{range .spec.containers[*]}{.image}{"\t"}{end}{end}'`: essa é a expressão JSONPath que define o formato de saída do comando. Ela usa a função range para iterar sobre todos os objetos items (ou seja, os Pods) retornados pelo comando kubectl get pods. Em seguida, exibe o nome do Pod `({.metadata.name})` seguido de uma tabulação `(\t)`, e itera sobre todos os contêineres `({range .spec.containers[*]})` dentro do Pod, exibindo a imagem usada por cada um deles `({.image})`. Por fim, insere uma quebra de linha `(\n)` e fecha o segundo range com `{end}{end}`.

Sim, eu sei, continua confuso!

Mas vou te contar um segredo, somente com o tempo e utilizando repetidas vezes, as coisas começam a ficar mais fáceis, então não desista! Pra trás, nem pra pegar impulso!

Ainda vamos falar com mais detalhes sobre como utilizar `metadata` para ter uma saída mais amigável e precisa.

#### Apagando o ReplicaSet

Para remover o `ReplicaSet` e todos os `Pods` que ele está gerenciando, basta executar o comando abaixo.

```bash
kubectl delete replicaset nginx-replicaset
```

&nbsp;

Caso você queira fazer isso utilizando o arquivo de manifesto, basta executar o comando abaixo.

```bash
kubectl delete -f nginx-replicaset.yaml
```

&nbsp;

Pronto, o nosso `ReplicaSet` foi removido e todos os `Pods` que ele estava gerenciando também foram removidos.

Durante a nossa sessão, nós já aprendemos como criar um `ReplicaSet` e como ele funciona, mas ainda temos muito o que aprender, então vamos continuar.

&nbsp;

### O DaemonSet

Já sabemos o que é um `Pod`, um `Deployment` e um `ReplicaSet`, mas agora é a hora de conhecermos mais um objeto do `Kubernetes`, o `DaemonSet`.

O `DaemonSet` é um objeto que garante que todos os nós do cluster executem uma réplica de um `Pod`, ou seja, ele garante que todos os nós do cluster executem uma cópia de um `Pod`.

O `DaemonSet` é muito útil para executar `Pods` que precisam ser executados em todos os nós do cluster, como por exemplo, um `Pod` que faz o monitoramento de logs, ou um `Pod` que faz o monitoramento de métricas.

Alguns casos de uso de `DaemonSets` são:

- Execução de agentes de monitoramento, como o `Prometheus Node Exporter` ou o `Fluentd`.
- Execução de um proxy de rede em todos os nós do cluster, como  `kube-proxy`, `Weave Net`, `Calico` ou `Flannel`.
- Execução de agentes de segurança em cada nó do cluster, como  `Falco` ou `Sysdig`.


Portanto, se nosso cluster possuir 3 nós, o `DaemonSet` vai garantir que todos os nós executem uma réplica do `Pod` que ele está gerenciando, ou seja, 3 réplicas do `Pod`.

Caso adicionemos mais um `node` ao cluster, o `DaemonSet` vai garantir que todos os nós executem uma réplica do `Pod` que ele está gerenciando, ou seja, 4 réplicas do `Pod`.

#### Criando um DaemonSet

Vamos para o nosso primeiro exemplo, vamos criar um `DaemonSet` que vai garantir que todos os nós do cluster executem uma réplica do `Pod` do `node-exporter`, que é um exporter de métricas do `Prometheus`.

Para isso, vamos criar um arquivo chamado `node-exporter-daemonset.yaml` e vamos adicionar o seguinte conteúdo.

```yaml
apiVersion: apps/v1 # Versão da API do Kubernetes do objeto
kind: DaemonSet # Tipo do objeto
metadata: # Informações sobre o objeto
  name: node-exporter # Nome do objeto
spec: # Especificação do objeto
  selector: # Seletor do objeto
    matchLabels: # Labels que serão utilizadas para selecionar os Pods
      app: node-exporter # Label que será utilizada para selecionar os Pods
  template: # Template do objeto
    metadata: # Informações sobre o objeto
      labels: # Labels que serão adicionadas aos Pods
        app: node-exporter # Label que será adicionada aos Pods
    spec: # Especificação do objeto, no caso, a especificação do Pod
      hostNetwork: true # Habilita o uso da rede do host, usar com cuidado
      containers: # Lista de contêineres que serão executados no Pod
      - name: node-exporter # Nome do contêiner
        image: prom/node-exporter:latest # Imagem do contêiner
        ports: # Lista de portas que serão expostas no contêiner
        - containerPort: 9100 # Porta que será exposta no contêiner
          hostPort: 9100 # Porta que será exposta no host
        volumeMounts: # Lista de volumes que serão montados no contêiner, pois o node-exporter precisa de acesso ao /proc e /sys
        - name: proc # Nome do volume
          mountPath: /host/proc # Caminho onde o volume será montado no contêiner
          readOnly: true # Habilita o modo de leitura apenas
        - name: sys # Nome do volume 
          mountPath: /host/sys # Caminho onde o volume será montado no contêiner
          readOnly: true # Habilita o modo de leitura apenas
      volumes: # Lista de volumes que serão utilizados no Pod
      - name: proc # Nome do volume
        hostPath: # Tipo de volume 
          path: /proc # Caminho do volume no host
      - name: sys # Nome do volume
        hostPath: # Tipo de volume
          path: /sys # Caminho do volume no host
```

&nbsp;

Eu deixei o arquivo comentado para facilitar o entendimento, agora vamos criar o `DaemonSet` utilizando o arquivo de manifesto.

```bash
kubectl apply -f node-exporter-daemonset.yaml
```

&nbsp;

Agora vamos verificar se o `DaemonSet` foi criado.

```bash
kubectl get daemonset
```

&nbsp;

Como podemos ver, o `DaemonSet` foi criado com sucesso.

```bash
NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
node-exporter   2         2         2       2            2           <none>          5m24s
```

&nbsp;

Caso você queira verificar os `Pods` que o `DaemonSet` está gerenciando, basta executar o comando abaixo.

```bash
kubectl get pods -l app=node-exporter
```

&nbsp;

Somente para lembrar, estamos utilizando o parâmetro `-l` para filtrar os `Pods` que possuem a label `app=node-exporter`, que é o caso do nosso `DaemonSet`.

Como podemos ver, o `DaemonSet` está gerenciando 2 `Pods`, um em cada nó do cluster.

```bash
NAME                  READY   STATUS    RESTARTS   AGE
node-exporter-k8wp9   1/1     Running   0          6m14s
node-exporter-q8zvw   1/1     Running   0          6m14s
```

&nbsp;

Os nossos `Pods` do `node-exporter` foram criados com sucesso, agora vamos verificar se eles estão sendo executados em todos os nós do cluster.

```bash
kubectl get pods -o wide -l app=node-exporter
```

&nbsp;

Com o comando acima, podemos ver em qual nó cada `Pod` está sendo executado.

```bash
NAME                                READY   STATUS    RESTARTS   AGE     IP               NODE                            NOMINATED NODE   READINESS GATES
node-exporter-k8wp9                 1/1     Running   0          3m49s   192.168.8.145    ip-192-168-8-145.ec2.internal   <none>           <none>
node-exporter-q8zvw                 1/1     Running   0          3m49s   192.168.55.68    ip-192-168-55-68.ec2.internal   <none>           <none>
```

&nbsp;

Como podemos ver, os `Pods` do `node-exporter` estão sendo executados em todos os dois nós do cluster.

Para ver os detalhes do `DaemonSet`, basta executar o comando abaixo.

```bash
kubectl describe daemonset node-exporter
```

&nbsp;

O comando acima vai retornar uma saída parecida com a abaixo.

```bash
Name:           node-exporter
Selector:       app=node-exporter
Node-Selector:  <none>
Labels:         <none>
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 2
Current Number of Nodes Scheduled: 2
Number of Nodes Scheduled with Up-to-date Pods: 2
Number of Nodes Scheduled with Available Pods: 2
Number of Nodes Misscheduled: 0
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=node-exporter
  Containers:
   node-exporter:
    Image:        prom/node-exporter:latest
    Port:         9100/TCP
    Host Port:    9100/TCP
    Environment:  <none>
    Mounts:
      /host/proc from proc (ro)
      /host/sys from sys (ro)
  Volumes:
   proc:
    Type:          HostPath (bare host directory volume)
    Path:          /proc
    HostPathType:  
   sys:
    Type:          HostPath (bare host directory volume)
    Path:          /sys
    HostPathType:  
Events:
  Type    Reason            Age   From                  Message
  ----    ------            ----  ----                  -------
  Normal  SuccessfulCreate  9m6s  daemonset-controller  Created pod: node-exporter-q8zvw
  Normal  SuccessfulCreate  9m6s  daemonset-controller  Created pod: node-exporter-k8wp9
```

&nbsp;

Na saída acima, podemos ver algumas informações bem importantes relacionadas ao `DaemonSet`, como por exemplo, o número de nós que o `DaemonSet` está gerenciando, o número de `Pods` que estão sendo executados em cada nó, etc.

Você ainda pode criar um `DaemonSet` utilizando o comando `kubectl create`, mas eu prefiro utilizar o arquivo de manifesto, pois assim eu consigo versionar o meu `DaemonSet`, mas caso você queira criar um `DaemonSet` utilizando o comando `kubectl create`, basta executar o comando abaixo.

```bash
kubectl create daemonset node-exporter --image=prom/node-exporter:latest --port=9100 --host-port=9100
```

&nbsp;

Ficaram faltando alguns parâmetros no comando acima, mas eu deixei assim para facilitar o entendimento, caso você queira ver todos os parâmetros que podem ser utilizados no comando `kubectl create daemonset`, basta executar o comando abaixo.

```bash
kubectl create daemonset --help
```

&nbsp;

Eu gosto de utilizar o `kubectl create` somente para criar um arquivo exemplo, para que eu possa me basear na hora de criar o meu arquivo de manifesto, mas caso você queira criar um manifesto para criar `DaemonSet` utilizando o comando `kubectl create`, basta executar o comando abaixo.

```bash
kubectl create daemonset node-exporter --image=prom/node-exporter:latest --port=9100 --host-port=9100 -o yaml --dry-run=client > node-exporter-daemonset.yaml
```

&nbsp;

Simples assim! Vou te explicar o que está acontecendo no comando acima.

- `kubectl create daemonset node-exporter` - Cria um `DaemonSet` chamado `node-exporter`.
- `--image=prom/node-exporter:latest` - Utiliza a imagem `prom/node-exporter:latest` para criar os `Pods`.
- `--port=9100` - Define a porta `9100` para o `Pod`.
- `--host-port=9100` - Define a porta `9100` para o nó.
- `-o yaml` - Define o formato do arquivo de manifesto como `yaml`.
- `--dry-run=client` - Executa o comando sem criar o `DaemonSet`, somente simula a criação do `DaemonSet`.
- `> node-exporter-daemonset.yaml` - Redireciona a saída do comando para o arquivo `node-exporter-daemonset.yaml`.

Ficou mais simples, certo?

#### Aumentando um node no cluster

Agora que já sabemos como criar um `DaemonSet`, vamos aumentar o número de nós do nosso cluster.

Nós estamos com duas réplicas nesse momento.

```bash
kubectl get nodes
```

&nbsp;

```bash
NAME                             STATUS   ROLES    AGE    VERSION
ip-192-168-55-68.ec2.internal    Ready    <none>   113m   v1.23.16-eks-48e63af
ip-192-168-8-145.ec2.internal    Ready    <none>   113m   v1.23.16-eks-48e63af
```

&nbsp;

Vamos aumentar o número de nós para 3.

Eu estou utilizando o `eksctl` para criar o cluster, então eu vou utilizar o comando `eksctl scale nodegroup` para aumentar o número de nós do cluster, mas caso você esteja utilizando outro gerenciador de cluster, você pode utilizar o comando que preferir para aumentar o número de nós do cluster.

```bash
eksctl scale nodegroup --cluster=eks-cluster --nodes 3 --name eks-cluster-nodegroup
```

&nbsp;

```bash
2023-03-11 13:31:48 [ℹ]  scaling nodegroup "eks-cluster-nodegroup" in cluster eks-cluster
2023-03-11 13:31:49 [ℹ]  waiting for scaling of nodegroup "eks-cluster-nodegroup" to complete
2023-03-11 13:33:17 [ℹ]  nodegroup successfully scaled
```

&nbsp;

Vamos verificar se o node foi adicionado ao cluster.

```bash
kubectl get nodes
```

&nbsp;

```bash
NAME                             STATUS   ROLES    AGE    VERSION
ip-192-168-45-194.ec2.internal   Ready    <none>   47s    v1.23.16-eks-48e63af
ip-192-168-55-68.ec2.internal    Ready    <none>   113m   v1.23.16-eks-48e63af
ip-192-168-8-145.ec2.internal    Ready    <none>   113m   v1.23.16-eks-48e63af
```

&nbsp;

Pronto, agora nós temos 3 nós no cluster.

Mas a pergunta que não quer calar é: O `DaemonSet` criou um `Pod` no novo nó?

Vamos verificar.

```bash
kubectl get pods -o wide -l app=node-exporter
```

&nbsp;

```bash
NAME                  READY   STATUS    RESTARTS   AGE   IP               NODE                             NOMINATED NODE   READINESS GATES
node-exporter-k8wp9   1/1     Running   0          20m   192.168.8.145    ip-192-168-8-145.ec2.internal    <none>           <none>
node-exporter-q8zvw   1/1     Running   0          20m   192.168.55.68    ip-192-168-55-68.ec2.internal    <none>           <none>
node-exporter-xffgq   1/1     Running   0          70s   192.168.45.194   ip-192-168-45-194.ec2.internal   <none>           <none>
```

&nbsp;

Parece que temos um novo `Pod` no nó `ip-192-168-45-194.ec2.internal`, mas vamos verificar se o `DaemonSet` está gerenciando esse nó.

```bash
kubectl describe daemonset node-exporter
```

&nbsp;

```bash
Desired Number of Nodes Scheduled: 3
Current Number of Nodes Scheduled: 3
Number of Nodes Scheduled with Up-to-date Pods: 3
Number of Nodes Scheduled with Available Pods: 3
Number of Nodes Misscheduled: 0
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
```

&nbsp;

Tudo em paz e harmonia, o `DaemonSet` está gerenciando o novo `Pod` no novo `node`.

















&nbsp;

### A sua lição de casa

A sua lição de casa é 

&nbsp;
## Desafio do Day-4

Não esqueça de

&nbsp;

## Final do Day-4

Durante o Day-4 você aprendeu 


&nbsp;
