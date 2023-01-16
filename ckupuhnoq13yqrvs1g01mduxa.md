# Virtualização nos Mac M1? Sim, com UTM

Como todos já sabem, a Apple resolveu inovar mais uma vez e nos trouxe novas máquinas com seu chip M1. Isso é legal? Sim, mas junto com ele, nos trouxe também problemas de compatibilidade, já que a arquitetura do chip M1 é arm64 e não o tradicional x86_64.

Por conta disto, soluções de virtualização já consolidadas no mercado, como o [VirtualBox](https://www.virtualbox.org/) não funcionam, já que este, funciona somente na arquitetura x86_64.

Porém, o mundo open source é lindo e graças a ele, temos uma solução! E esta solução é o [UTM](https://mac.getutm.app).

O [UTM](https://mac.getutm.app) foi originalmente criado para virtualização em iPhones, porém, como os novos MacBooks também usam o mesmo chip M1, foi criado a versão desktop também.

A instalação do [UTM](https://mac.getutm.app) é muito simples, bastando baixar o .dmg a partir do site ou do repositório do projeto no [GitHub](https://github.com/utmapp/UTM), executar e copiar o UTM para a pasta Applications. Uma alternativa interessante também é a instalação pela Mac Apple Store.

No próprio site do [UTM](https://mac.getutm.app) tem uma galeria, onde você encontra várias opções de distros linux para instalação e até mesmo algumas versões do Windows pode ser executadas. Você pode consultar tudo isto em https://mac.getutm.app/gallery/

Um projeto lindo desses merece uma star no repositório deles no GitHub, https://github.com/utmapp/UTM.

#### Mas Joubert, existe alguma alternativa?

Então, até tem, o [Parallels Desktop for Mac](https://www.parallels.com/blogs/parallels-desktop-apple-silicon-mac/), porém, não é uma solução open source e você precisa ter uma conta para usar, o que para mim não faz o menor sentido.

Até o momento, só conheço essas duas opções que realmente funcionam no Mac M1, caso eu conheça mais soluções ou novidades, volto com novos posts sobre o assunto.