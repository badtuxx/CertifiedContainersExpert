# Descomplicando o Kubernetes - Expert Mode

## DAY-6
&nbsp;

&nbsp;
### Conteúdo do Day-6


&nbsp;
### O que iremos ver hoje?

Hoje é dia de falar sobre dois resources muito importantes para o Kubernetes, o `StatefulSet` e o `Service`.
Vamos mostrar como criar e administrar `StatefulSets` para que você possa criar aplicações que precisam manter a identidade do `Pod` e persistir dados em volumes locais. Caos bem comuns de uso de `StatefulSets` são bancos de dados, sistemas de mensageria e sistemas de arquivos distribuídos.

Outra peça super importante que iremos falar hoje é sobre o `Service`. O `Service` é um objeto que permite que você expor uma aplicação para o mundo externo. Ele é responsável por fazer o balanceamento de carga entre os `Pods` que estão sendo expostos e também por fazer a resolução de nomes DNS para os `Pods` que estão sendo expostos.
Temos diversos tipos de `Services` e iremos falar sobre eles hoje.

Então esteja preparado para uma viajem muito interessante sobre esses dois recursos que são muito importantes para o Kubernetes e para o dia a dia de uma pessoa que trabalha com ele.

### O que é um StatefulSet?

Primeira coisa que temos que ter em mente quando falamos sobre `StatefulSets` no Kubernetes, é que muito do que nós veremos aqui será bem parecido com o conceito de `Deployment`, porém com algumas diferenças e vou listar elas aqui para vocês.

`StatefulSets` são usados para aplicações que precisam manter a identidade do `Pod` e persistir dados em volumes locais, dito isso é bem comum de se ver `StatefulSets` sendo usados para bancos de dados, sistemas de mensageria e sistemas de arquivos distribuídos.

Outra caracteristica muito importante do `StatefulSets` é o melhor gerenciamento no momento de criação e remoção de `Pods`. Isso é feito através de um mecanismo de controle interno que garante que os `Pods` sejam criados e removidos na ordem correta, na ordem que você definiu no `StatefulSet`.

Outra caracteristica importante é que os `Pods` criados por um `StatefulSet` são criados com um nome estático, ou seja, o nome do `Pod` é baseado no nome do `StatefulSet` e no índice do `Pod` dentro do `StatefulSet`. Isso é muito importante para que você possa fazer referência a um `Pod` específico dentro do `StatefulSet` e também para que você possa fazer referência a um `Pod` específico dentro do `StatefulSet` através de um `Service`. Com isso, mesmo que o `Pod` seja removido e criado novamente, ele continuará com o mesmo nome.















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
