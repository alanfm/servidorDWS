# Instalação de um servidor Debian para DNS/Web/Samba

## Instalção do Servidor

1. Baixar a imagem de instação do Sistema Operacional.
    + [ISO Debina 10.2](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-10.2.0-amd64-netinst.iso)
2. Criar um pendrive bootavel e configurar o computador para dar boot pelo pendrive.
3. Ao iniciar o sistema pelo pendrive siga os seguintes passos:
    + Assim que o sistema iniciar selecione a opção ```Install```;
    + Selecione a linguagem de instação do Sistema Operacional - nesse caso Portuguese (Brazil) -  Português do Brasil;
    + Selecione a localidade - Brasil;
    + Selecione o mapa do teclado - Português Brasileiro;
        - Nesse o memento o sistema buscará uma conexão com a internet - nesse caso pelo DHCP;
    + Agora o sistema pedirá para informar o nome da máquina - Utilizo ```ServidorDWS```;
    + Informe o nome do dominio - Utilizo ```baturite.ifce```;
    + Informe a senha de administrador do sistema (root) e sua confirmação - Utilizo a senha padrão da TI;
    + Informe o nome completo do usuário padrão do sistema - Utilizo ```Coordenação de Tecnologia da Informação```;
    + Informe o nome de usuário (apenas uma palavra) - Utilizo ```cti```;
    + Informe a senha e confirmação de senha do usuário padrão - Utilizo a mesma senha padrão da TI;
    + Na configuração do relógio selecione Ceará;
        - Após esses passos o sistema fará algumas configurações automáticas;
    + No particionamento de disco selecione a opção ```Assistido - usar o disco inteiro```;
    + Selecione o disco (HD);
    + Selecione a opção ```Todos os arquivos em uma particão (para iniciantes)```;
    + Selecione a opção ```Finalizar o particionamento e escrever as mudanças no disco```;
    + Selecione a opção ```Sim``` para escrever as partições no disco;
        - Agora sistema começará a sua instalação no disco.
    + Na sessão de gerenciador de pacotes (Ler outro CD/DVD) selecione ```Não```;
    + Selecione Brasil;
    + Selecione ```deb.debian.org```;
    + Na configuração de proxi deixe em branco;
    + Participar do concurso de utlização de pacotes? Selecione ```Não```;
    + Na seleção de software selecione as seguintes opções:
        - ```Servidor SSH```
        - ```Utilitários de sistema padrão```
    + Selecione ```Sim```;
    + Selecione ```/dev/sda ("nome do disco")```;
    + E por ultimo ```Continuar```;

## Configuração do Sistema

1. Primeiramente deverá ser setado o IP como estático.
    + Verifique o IP atual do computador através do comando:
        - ```$ ip a```;
    + Edite o arquivo ```/etc/network/interfaces``` através do comando: ```# nano /etc/network/interfaces```;
        - Deixe o arquivo com as seguintes linhas (Lembrando que o nome da interface pode ser diferente em cada maquina e as configurações de IP também);
    ```
        # This file describes the network interfaces available on your system
        # and how to activate them. For more information, see interfaces(5)
        
        source /etc/network/interfaces.d/*
        
        # The loopback network interface
        auto lo
        iface lo inet loopback
        
        # The primary network interface
        auto enp0s3
        iface enp0s3 inet static
            address 192.168.0.2
            netmask 255.255.255.0
            network 192.168.0.0
            broadcast 192.168.0.255
            gateway 192.168.0.1
    ```
    + Pressione ```Ctrl+o``` para salvar e ```Ctrl+x``` para sair.

## Instalando o Bind9 (Servidor de DNS)
1. Usei o tutorial da Digital Ocean (https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-debian-9) para fazer a instalação e configuração do Bin9.
    + Até o passo "Setting Bind to IPv4 Mode" é igual ao tutorial, a partir de "Configuring the Primary DNS Server" colocar as configurações que serão usadas no campus.
    + No caixa onde pede para criar uma acl(/etc/bind/named.conf.options - 1 de 3), coloca-se apenas o ip do servidor (10.100.111.254), já que todos os serviços vão rodar na mesma maquina.
    
    ```
        acl "trusted" {
            10.100.111.254;
        };
    ```
    
    + Na caixa que pedi para adicionar as algumas linhas, coloca-se o ip do servidor dns (10.100.111.254) em ``` listen-on { 10.100.111.254 } ``` e em ```forwarders``` colocar na primeira linha o ip de DNS mais rápido fora do campus (uso o 1.1.1.1).
    + Na seção "Configuring the Local File - /etc/bind/named.conf.local — 1 of 2" trocamos o endereço do exemplo "nyc3.example.com" por "baturite.ifce" em todas as ocorrencias. Ficará como no exemplo abaixo.
    ```
        zone "baturite.ifce" {
            type master;
            file "/etc/bind/zones/db.baturite.ifce"; # zone file path
        };
    ```
    + Na box "/etc/bind/named.conf.local — 2 of 2" trocaremos os trechos "128.10" por "10.100" que é a classe de IP que usamos no campus.
    ```
        zone "10.100.in-addr.arpa" {
            type master;
            file "/etc/bind/zones/db.10.100";  # 10.100.96.0/16 subnet
        };
    ```
    
    + No final da seção "Creating the Forward Zone File" o arquivo deve ficar desse jeito
    
    ```
        $TTL    604800
        @       IN      SOA     baturite.ifce. admin.baturite.ifce. (
                          3     ; Serial
                     604800     ; Refresh
                      86400     ; Retry
                    2419200     ; Expire
                     604800 )   ; Negative Cache TTL
        ;
        ; name servers - NS records
             IN      NS      ns.baturite.ifce.

        ; name servers - A records
        ns.baturite.ifce.              IN      A      10.100.111.254

        ; 10.128.0.0/16 - A records
        reservas.baturite.ifce.        IN      A      10.100.111.254
        frequencias.baturite.ifce.     IN      A      10.100.111.254
    ```
    + Em "Creating the Reverse Zone File(s)" o arquivo deve ficar no final como abaixo. 
    ```
        $TTL    604800
        @       IN      SOA     baturite.ifce. admin.baturite.ifce. (
                                      3         ; Serial
                                 604800         ; Refresh
                                  86400         ; Retry
                                2419200         ; Expire
                                 604800 )       ; Negative Cache TTL
        ; name servers
              IN      NS      ns.baturite.ifce.

        ; PTR Records
        10.100 IN      PTR     ns.baturite.ifce.    ; 10.100.111.254
        10.100 IN      PTR     reservas.baturite.ifce.
        10.100 IN      PTR     frequencias.baturite.ifce.
    ```
    + Vale ressaltar que sempre que for inserido um novo subdominio é necessário acrescenta-lo no arquivo de zona e zona reverso.
    + Faça os teste de arquivos e sitaxe colocados no exemplo trocando para o nome dos arquivos e zonas criadas para verificação de erros.
    + Não é para fazer a seção "Configuring the Secondary DNS Server", pois usaremos apenas um servidor de DNS.
    
    
## Instalando o Git (Serviço de versionamento)
1. O git é um serviço de versionamento para pegar os sistemas que estão no GitHub
    + ```# apt install git -y```
    
## Instalando o Docker (Serviço de conteiners)
1. Para instalar o docker é bem simples, primeiro precisamos instalar o ```Curl``` para baixar um arquivo para instalação automática do Docker.
    + ```# apt install curl -y```
    + Usei a documentação do Docker para fazer esse procedimento (https://docs.docker.com/engine/install/debian/).
        - Na seção "Install using the convenience script" tem todos os procedimentos necessários. Porém basta executar as linhas de comando abaixo.
        - ```# curl -fsSL https://get.docker.com -o get-docker.sh```
        - ```# sh get-docker.sh```
        - ```# usermod -aG docker cti```
        
## Criando os containers para rodas os sistemas
1. Com o git faremos o clone do projeto ```alanfm/docker``` através do comando ```$ git clone https://github.com/alanfm/docker.git```
    + Quando executado o comando será criado um diretório chamado docker. Entre no diretório ```cd docker/```.
    + Faça a cópia do arquivo ```exemplo.env``` com o comando ```$ cp exemplo.env .env```.
    + Será cria um arquivo chamado ```.env``` nesse arquito estarão todas as configurações para os conteiner que serão criado.
    + Abra o arquivo com um editor de texto. ```$ nano .env```. Navegue pelas linhas usando as setas do teclado e faça as modificações necessárias para rodar no servidor, como mudar as senhas e usuários para nomes mais seguros.
    + Execute o comando ```$ docker-compose up -d```. Nessa parte levará algum tempo, pois vai baixar as imagens para uso dos conteiners e será feito a compilação de alguns módulos.
    
