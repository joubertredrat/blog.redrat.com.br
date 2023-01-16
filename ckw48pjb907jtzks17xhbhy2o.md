# Pequena dica para o dia a dia, git clone-github repository

Hoje venho trazer uma dica que pode ser útil para vocês, um comando customizado para git que eu criei, o `git clone-github repository`.

### Pra que esse comando Joubert?

Bom, deixa eu explicar como gosto de organizar projetos no meu computador.

Na pasta home do meu usuário, eu tenho uma pasta chamada `source`, que é a pasta raiz onde eu baixo código, faço clone de projetos, etc, como vocês podem ver abaixo.

Eu gosto de usar esta forma de organizar código inclusive em todo computador que eu programo, tornando fácil identificar de qual projeto a pasta pertence.

![Projetos dentro da pasta source](https://cdn.hashnode.com/res/hashnode/image/upload/v1637192528570/zbOuyiTTQ.png)
Para este exemplo, eu criei a pasta `personal`, `picpay` e `unico_idtech`, ambas estão vazias.

Uma das pastas é a `opensource`, que o próprio nome já diz, contém projetos open source.

![Projetos dentro da pasta opensource](https://cdn.hashnode.com/res/hashnode/image/upload/v1637192705437/tYbnAekDl.png)

Dentro da pasta `opensource` existe a `joubertredrat` e `joubertredrat-tests` que são minhas pastas de trabalho relacionado a meus projetos open source, porém, além delas, eu tenho a pasta `other` que é justamente onde eu baixo projetos open source eu uso como fonte de conhecimento.

E é ai que vem a utilidade do comando `git clone-github repository`.

### Como assim Joubert?

Como na pasta `source/opensource/other` tem vários projetos de varias pessoas ou comunidades, as vezes torna difícil saber qual é a origem do projeto, como você pode ver abaixo.

![Descobrindo de quem é o projeto compose](https://cdn.hashnode.com/res/hashnode/image/upload/v1637195082814/K31gChrxn.png)

Ou seja, eu tive que entrar na pasta `compose` e executar `git remote -v` para descobrir que projeto é este, um projeto do docker. Com poucas pastas pode não ser um problema, mas quando passa das dezenas, pode ficar mais complicado achar o projeto que você deseja.

Com isto, eu criei um comando customizado do git para que sempre que for realizado o clone de um repositório do Github, ele faça para uma pasta contendo o nome do usuário ou organização como prefixo da pasta do clone.

Abaixo, um exemplo do mesmo repositório acima, com o clone feito pelo `git clone-github`.

![Clonando repositório com git clone-github](https://cdn.hashnode.com/res/hashnode/image/upload/v1637195434550/NgimScc08.png)

Com isto, fica mais fácil e organizado navegar por esta pasta e identificar facilmente projetos, usuários e organizações. Como nos exemplos abaixo.

* https://github.com/flutter/samples
* https://github.com/docker/cli
* https://github.com/docker/compose

Fica muito mais fácil você ver uma pasta `flutter__samples` e entender que é um projeto do flutter do que simplesmente `samples`, não acha?

### Como isto é possível e como fazer isso Joubert?

Isto é possível a comandos customizáveis que você pode adicionar ao git. Um exemplo disto pode ser visto [neste artigo](https://gitbetter.substack.com/p/automate-repetitive-tasks-with-custom).

Com isto, basta criar o script com o nome que você deseja e tendo o prefixo git, no meu caso `git-clone-github`, colocar permissão de modo executável `+x` e colocar este script em uma pasta que esteja no `$PATH`.

Abaixo o snippet do script do `git-clone-github`.

%[https://gist.github.com/joubertredrat/7ed396c410a8860689c8022c224f98a2]

Feito isto, está pronto, basta executar o seu comando `git clone-github repository`.

### Alternativa

Após ter postado este artigo no Twitter, o Wanderson Camargo [@wandersonwhcr](https://twitter.com/wandersonwhcr) postou uma alternativa e solução interessante que gostei tanto que achei interessante postar aqui também, como nos exemplos abaixo.

```bash
git clone {https://github.com/,}docker/compose
git clone {git@github.com:,}docker/compose
``` 

Essa alternativa é interessante pois diferente do `git clone-github` que somente aceita url http do GitHub, o comando acima permite usar tanto url, quanto git por ssh.

Outra diferença é que o clone é feito em pastas e subpastas, ou seja, o comando acima irá fazer o clone na pasta `docker/compose`, enquanto o `git clone-github` faz o clone na pasta `docker__compose`.

Por fim, a alternativa acima funciona nos shells mais utilizados, como sh, bash, zsh e [fish](https://fishshell.com/).

Até a próxima.

