# Descomplicando o Kubernetes - Expert Mode

Esse treinamento foi desenhado para capacitar a pessoa estudante ou a pessoa profissional de TI a trabalhar com o Kubernetes em ambientes criticos.

O Treinamento é composto por material escrito, aulas gravadas em vídeo e aulas ao vivo. Durante o treinamento a pessoa será testada de forma prática, sendo necessário completar desafios reais para dar continuidade no treinamento.

O foco do treinamento é capacitar a pessoa para trabalhar com Kubernetes de maneira eficiente e totalmente preparada para trabalhar em ambientes críticos que utilizam containers.


## Conteúdo
&nbsp;
- [DAY-2](#day-2)
    - [O quê preciso saber antes de começar?](day-1/README.md#o-quê-preciso-saber-antes-de-começar)
    - [Inicio da aula do Day-1](day-1/README.md#inicio-da-aula-do-day-1)
        - [Qual a distro GNU/Linux que devo usar?](day-1/README.md#qual-a-distro-gnu/linux-que-devo-usar?)
        - [Alguns sites que devemos visitar](day-1/README.md#alguns-sites-que-devemos-visitar)
        - [O Container Engine](day-1/README.md#o-container-engine)
        - [OCI - Open Container Initiative](day-1/README.md#oci---open-container-initiative)
        - [O Container Runtime](day-1/README.md#o-container-runtime)
        - [O que é o Kubernetes?](day-1/README.md#o-que-é-o-kubernetes?)
            - [Arquitetura do k8s](day-1/README.md#arquitetura-do-k8s)
        - [Instalando e customizando o Kubectl](day-1/README.md#instalando-e-customizando-o-kubectl)
            - [Instalação do Kubectl no GNU/Linux](day-1/README.md#instalação-do-kubectl-no-gnu/linux)
            - [Instalação do Kubectl no MacOS](day-1/README.md#instalação-do-kubectl-no-macos)
            - [Instalação do Kubectl no Windows](day-1/README.md#instalação-do-kubectl-no-windows)
            - [Customizando o kubectl](day-1/README.md#customizando-o-kubectl)
            - [Auto-complete do kubectl](day-1/README.md#auto-complete-do-kubectl) 
            - [Criando um alias para o kubectl](day-1/README.md#criando-um-alias-para-o-kubectl)
        - [Criando um cluster Kubernetes](day-1/README.md#criando-um-cluster-kubernetes)
            - [Criando o cluster em sua máquina local](day-1/README.md#criando-o-cluster-em-sua-máquina-local)
                - [Minikube](day-1/README.md#minikube)
                    - [Requisitos básicos para o Minikube](day-1/README.md#requisitos-básicos-para-o-minikube)
                    - [Instalação do Minikube no GNU/Linux](day-1/README.md#instalação-do-minikube-no-gnu/linux)
                    - [Instalação do Minikube no MacOS](day-1/README.md#instalação-do-minikube-no-macos)
                    - [Instalação do Minikube no Microsoft Windows](day-1/README.md#instalação-do-minikube-no-microsoft-windows)
                    - [Iniciando, parando e excluindo o Minikube](day-1/README.md#iniciando,-parando-e-excluindo-o-minikube)
                    - [Ver detalhes sobre o cluster](day-1/README.md#ver-detalhes-sobre-o-cluster)
                    - [Descobrindo o endereço do Minikube](day-1/README.md#descobrindo-o-endereço-do-minikube)
                    - [Acessando a máquina do Minikube via SSH](day-1/README.md#acessando-a-máquina-do-minikube-via-ssh)
                    - [Dashboard do Minikube](day-1/README.md#dashboard-do-minikube)
                    - [Logs do Minikube](day-1/README.md#logs-do-minikube)
                    - [Remover o cluster](day-1/README.md#remover-o-cluster)
                - [Kind](day-1/README.md#kind)
                    - [Instalação no GNU/Linux](day-1/README.md#instalação-no-gnu/linux)
                    - [Instalação no MacOS](day-1/README.md#instalação-no-macos)
                    - [Instalação no Windows](day-1/README.md#instalação-no-windows)
                    - [Instalação no Windows via Chocolatey](day-1/README.md#instalação-no-windows-via-chocolatey)
                    - [Criando um cluster com o Kind](day-1/README.md#criando-um-cluster-com-o-kind)
                    - [Criando um cluster com múltiplos nós locais com o Kind](day-1/README.md#criando-um-cluster-com-múltiplos-nós-locais-com-o-kind)
        - [Primeiros passos no k8s](day-1/README.md#primeiros-passos-no-k8s)
            - [Verificando os namespaces e pods](day-1/README.md#verificando-os-namespaces-e-pods)
            - [Executando nosso primeiro pod no k8s](day-1/README.md#executando-nosso-primeiro-pod-no-k8s)
            - [Expondo o pod e criando um Service](day-1/README.md#expondo-o-pod-e-criando-um-service)
        - [Limpando tudo e indo para casa](day-1/README.md#limpando-tudo-e-indo-para-casa)
- [DAY-2](day-2/README.md#day-2)
  - [O que iremos ver hoje?](day-2/README.md#o-que-iremos-ver-hoje)
    - [O que é um Pod?](o-que-e-um-pod?)
    - [Criando um Pod](day-2/README.md#criando-um-pod)
    - [Visualizando detalhes sobre os Pods](day-2/README.md#visualizando-detalhes-sobre-os-pods)
    - [Removendo um Pod](day-2/README.md#removendo-um-pod)
    - [Criando um Pod através de um arquivo YAML](day-2/README.md#criando-um-pod-atraves-de-um-arquivo-yaml)
    - [Visualizando os logs do Pod](day-2/README.md#visualizando-os-logs-do-pod)
    - [Criando um Pod com mais de um container](day-2/README.md#criando-um-pod-com-mais-de-um-container)
  - [Os comandos `attach` e `exec`](day-2/README.md#os-comandos-attach-e-exec)
  - [Criando um container com limites de memória e CPU](day-2/README.md#criando-um-container-com-limites-de-memoria-e-cpu)
  - [Adicionando um volume EmptyDir no Pod](day-2/README.md#adicionando-um-volume-emptydir-no-pod)

&nbsp;


## O treinamento Descomplicando o Kubernetes - Expert Mode

Pensamos em fazer um treinamento realmente prático onde a pessoa consiga aprender os conceitos e teoria com excelente didática, utilizando exemplos e desafios práticos para que você consiga executar todo o conhecimento adquirido. Isso é muito importante para que você consiga fixar e explorar ainda mais o conteúdo do treinamento.
E por fim, vamos simular algumas conversas para que fique um pouco mais parecido com o dia-a-dia no ambiente de trabalho.

Durante o treinamento vamos passar por todos os tópicos importantes do Kubernetes, para que no final do treinamento você possua todo conhecimento e também toda a segurança para implementar e administrar o Kubernetes em ambientes críticos e complexos.

Estamos prontos para iniciar a nossa viagem?
&nbsp;

### O conteúdo programático

O conteúdo ainda será ajustado, e no final do treinamento teremos o conteúdo completo.


&nbsp;


### Como adquirir o treinamento?

Para adquirir o treinamento [Descomplicando o Kubernetes](https://www.linuxtips.io/) você deverá ir até a loja da [LINUXtips](https://www.linuxtips.io/).

&nbsp;

## A ideia do formato do treinamento

Ensinar Kubernetes de uma forma mais real, passando todo o conteúdo de forma prática e trazendo uma conexão com o ambiente real de trabalho.

Esse é o primeiro treinamento sobre Kubernetes de forma realmente prática, da vida real. Pois entendemos que prática é o conjunto de entendimento sobre um assunto, seguido de exemplos reais que possam ser reproduzidos e conectando tudo isso com a forma como trabalhamos.

Assim a definição de prática passa a ser um focada em o conhecimento da ferramenta e adicionando a realidade de um profissional no seu dia-a-dia aprendendo uma nova tecnologia, uma nova ferramenta.

Prepare-se para um novo tipo de treinamento, e o melhor, prepare-se para um novo conceito sobre treinamento prático e de aprendizado de tecnologia.
&nbsp;

### As pessoas (personagens) no treinamento

Temos algumas pessoas que vão nos ajudar durante o treinamento, simulando uma dinâmica um pouco maior e ajudando na imersão que gostaríamos. 


Ainda estamos desenvolvendo e aprimorando os personagens e o enredo, portanto ainda teremos muitas novidades.

&nbsp;

#### A Pessoa_X

A Pessoa_X é uma das pessoas responsáveis pela loja de meias Strigus Socket, que está no meio da modernização de seu infra e das ferramentas que são utilizadas.

Segundo uma pessoa que já trabalhou com a Pessoa_X, ela é a pessoa que está sempre procurando aprender para inovar em seu ambiente. Normalmente é através dela que surgem as novas ferramentas, bem como a resolução de um monte de problemas.

O nível de conhecimento dela é sempre iniciante quando ela entra em um novo projeto, porém ao final dele, ela se torna uma especialista e com uma boa experiência prática, pois ela foi exposta a diversas situações, que a fizeram conhecer a nova tecnologia muito bem e se sentindo muito confortável em trabalhar no projeto.

Pessoa_X, foi um prazer fazer essa pequena descrição sobre você! 

Seja bem-vinda nesse novo projeto e espero que você se divirta como sempre! 

Lembre-se sempre que eu, Jeferson, estarei aqui para apoiar você em cada etapa dessa jornada! Eu sou o seu parceiro nesse projeto e tudo o que você precisar nessa jornada! Bora!

&nbsp;

#### A Pessoa_Lider_X

Iremos criando a personalidade dessa pessoa durante o treinamento.
O que já sabemos é que ela é a pessoa líder imediata da Pessoa_X, e que irá demandar a maioria das tarefas. E tem como o esteriótipo um líder meio tosco.
&nbsp;

#### A Pessoa_Diretora_X

Líder imediato da Pessoa_Lider_X e que tem um sobrinho 'jênio' e que está ali, dando os seus pitacos no setor de tecnologia, por que ele 'mereceu', entendeu?

&nbsp;

#### A Pessoa_RH_X

A pessoa responsável pelo RH da empresa, no decorrer do treinamento vamos faz
endo a história e características dela.
&nbsp;


## Vamos começar?

Agora que você já conhece mais detalhes sobre o treinamento, acredito que já podemos começar, certo?

Lembrando que o treinamento está disponível na plataforma da escola da LINUXtips, que não é o mesmo endereço da [loja](https://www.linuxtips.io/), para acessar a escola [CLIQUE AQUI](https://www.linuxtips.io).
&nbsp;


### O conteúdo

Como o treinamento é dividido em **days**, acesse o diretório correto para acessar o conteúdo completar do treinamento do **day** em que vc está estudando.

- [ACESSE O DAY-1](day-1/README.md)

**Bons estudos!**

