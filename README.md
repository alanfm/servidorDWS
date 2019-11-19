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
        - Deixe o arquivo com as seguintes linhas;
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
