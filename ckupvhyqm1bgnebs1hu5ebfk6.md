# Rodando projetos x86_64 no Mac M1 (arm64) com UTM

Desde que a Apple publicou sobre os novos Macs com o chip M1, houve um grande esforço para que vários aplicativos e programas pudessem rodar na nova arquitetura, que é arm64, diferente da tradicional arquitetura x86_64 dos processadores Intel, AMD e outros, tanto de empresas donas dos softwares proprietários, quanto de comunidades de projetos open source.

Muito deste ecossistema tem suporte nos Mac M1, porém, alguns projetos ainda apresentam problemas, como alguns projetos baseados em QEMU ou imagens de docker que ainda não tem suporte multi arquitetura, por exemplo.

Uma forma de poder contornar essa situação é justamente poder executar esses projetos como arquitetura x86_64, e é ai que a virtualização pode nos ajudar.

Para virtualização, iremos usar o UTM, que já escrevi sobre neste link,  [Virtualização nos Mac M1? Sim, com UTM](https://blog.redrat.com.br/virtualizacao-nos-mac-m1-sim-com-utm). 

Então, vou assumir que você já tenha o UTM instalado no seu Mac M1, então vamos lá.

Para este artigo, vamos usar o Ubuntu 20.04 server, então você pode fazer o download no site da Canonical, ou direto no link abaixo, porém, você pode usar a distro de sua preferência.

https://releases.ubuntu.com/20.04/ubuntu-20.04.3-live-server-amd64.iso

### Criando a máquina virtual

* Vamos criar nossa máquina virtual clicando no ícone "+" e depois em "Start from Scratch".

![Criando nova máquina virtual no UTM](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150249374/piTCTAElC.png)

* Na aba "Information" você vai configurar o nome e o ícone da máquina virtual conforme desejar.

![Configurando informações da máquina virtual](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150325368/oOlpI_QOa.png)

* Na aba "System" vem a parte importante, em "Architecture", selecione a opção "x86_64", a opção "System" logo abaixo será alterado automaticamente. Na opção "Memory" você vai selecionar a quantidade de memória que a máquina virtual deverá ter. O mínimo para conseguir rodar é 768Mb, porém, recomendo fortemente que coloque 1Gb ou mais.

![Configurando arquitetura e memória da máquina virtual](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150386673/C1lqrn7Pm.png)

* Na aba "Drives", você irá criar um disco rígido e uma unidade de CD/DVD. Eu recomendo fortemente criar um disco com pelo menos 30 Gb ou mais.

![Criação de disco rígido da máquina virtual](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150470188/MK8cRY8lf.png)
![Criação de unidade de CD/DVD da máquina virtual](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150507715/9XMv6LelP.png)
![Disco rígido e unidade de CD/DVD da máquina virtual](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150566267/ORboTs0DtN.png)

* Na aba "Network", você irá criar um redirecionamento de portas, para que possamos acessar a máquina via SSH.

![Adicionando redirecionamento de portas da máquina virtual](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150715976/Q6hq2CiB4.png)
![Redirecionamento de portas da máquina virtual](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150745130/gD8H3yKGZ.png)

Vale lembrar que você pode fazer redirecionamento de outras portas que desejar.

* Por fim, só clicar em "Save" e pronto, sua máquina virtual estará criada.

![Máquina virtual criada](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150837172/fo3AB31H9.png)

* Na parte inferior das informações da máquina virtual existe um select na opção de CD/DVD, você irá selecionar a ISO da distro que você escolheu, o que no nosso caso, é o Ubuntu Server 20.04.

![Máquina virtual com a ISO da distro selecionada](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150926810/j2OdvDTPJ.png)

* Agora é só executar a máquina virtual e instalar a distro linux.

![Instalando linux na máquina virtual](https://cdn.hashnode.com/res/hashnode/image/upload/v1634150998330/Ro8c7uWmm.png)

### Testando a máquina virtual

Agora que você instalou sua máquina virtual, basta acessar ela por SSH e você verá que na máquina virtual, estará rodando um sistema x86_64, como na imagem abaixo.

![Terminal ssh na máquina virtual](https://cdn.hashnode.com/res/hashnode/image/upload/v1634151056368/lMm0Ard0z.png)

### Hora da prova de fogo

Para a o exemplo da prova de fogo estou usando o docker e o container de um projeto chamado Kafdrop, que basicamente é uma Web UI para Kafka feito em Java e que também merece uma estrela no GitHub, ~~mesmo que tenha sido feito em java haha~~ o repositório do projeto está abaixo. 

https://github.com/obsidiandynamics/kafdrop

Como o container deste projeto ainda não tem suporte a multi arquitetura, tentamos executar ele como se fosse `linux/amd64`, em outros projetos até funciona, mas neste não.

Como pode ser visto na imagem abaixo, no primeiro console ocorreu o erro `qemu: uncaught target signal 11 (Segmentation fault) - core dumped`, pois mesmo tentando forçar x86_64 na arch arm64, neste caso não funciona.

Porém, no segundo console o projeto executou normalmente, pois se trata de um container x86_64 rodando em uma distro x86_64.

![container executando no macOS e linux](https://cdn.hashnode.com/res/hashnode/image/upload/v1634151136436/ClVgGJpOq.png)

Ou seja, é perfeitamente possível executar projetos x86_64 mesmo estando com um Mac M1.

Então é isso pessoas, espero ter ajudado com este artigo e se tiverem mais dúvidas, fiquem a vontade para comentar por aqui mesmo ou me caçando em algum lugar da internet :)