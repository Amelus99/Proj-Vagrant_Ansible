### Disciplina: *Administração de Sistemas Abertos*
### Aluno(a): *Maria Isabel Saturnino*  Matrícula: *20211380035*
### Aluno(a): *Samuel Araújo Cabral e Silva* Matrícula: *20242380040*
### Professor: *Pedro Filho*
#                                    
# Projeto de Infraestrutura com Vagrant e Ansible
[!NOTE]
# 1. Introdução
Grandes avanços no mundo digital tornam evidente que o uso de ferramentas de automação e virtualização é essencial para aumentar a eficiência e reduzir significativamente os erros. O Vagrant e o Ansible são exemplos de tecnologias amplamente utilizadas nesse contexto.

Neste projeto, analisaremos a integração desses dois componentes, explicando de forma breve suas funcionalidades individuais e como trabalham juntos.

# 2. O que é Vagrant?
Vagrant é uma ferramenta de código aberto que permite a criação e configuração rápida de ambientes de desenvolvimento virtualizados. Com Vagrant, é possível:

Provisionar Ambientes: Criar máquinas virtuais de maneira automática e reprodutível.
Isolar Ambientes: Garantir que um ambiente de desenvolvimento não interfira em outro, usando máquinas virtuais isoladas.
Facilidade de Compartilhamento: Utilizando arquivos de configuração (Vagrantfile), outros desenvolvedores podem replicar o mesmo ambiente em suas máquinas com um simples comando.

# 3. O que é o Ansible?
O Ansible é uma ferramenta de automação open source escrita em Python. Ele é amplamente utilizado para o gerenciamento de configurações, permitindo a execução de múltiplas tarefas em diferentes ambientes. Trabalha sem a necessidade de agentes, utilizando o protocolo SSH para realizar os gerenciamentos.

Quase todos os parâmetros do Ansible são baseados em arquivos .yaml, como playbooks e roles. No entanto, alguns inventários não dependem exclusivamente desse formato e podem ser escritos em outras opções, como JSON.

A partir de palavras-chave e comandos (módulos e parâmetros), o Ansible possibilita o desenvolvimento de todos os processos de automação de forma eficiente e escalável.

# 4. Projeto
   ## Vagrantfile 
   O arquivo Vagrantfile configura uma máquina virtual baseada na box Debian Bookworm (64 bits) usando o VirtualBox como provedor. A VM é nomeada como "p01_isabel", possui 1 GB de memória RAM, e está configurada para usar uma rede privada com o IP 192.168.57.10. Além disso, são criados e conectados três discos adicionais de 10 GB cada, configurados na controladora SATA. O provisionamento da máquina é realizado com o Ansible, utilizando o arquivo playbook.yml para automatizar tarefas pós-criação.

         # -*- mode: ruby -*-
      # vi: set ft=ruby :
      # Before execute this Vagrant File, Type the command below in your Linux Terminal
      #   export VAGRANT_EXPERIMENTAL="disks"
      
      # Vagrantfile configurado para provisionamento da VM conforme especificado no projeto
      Vagrant.configure("2") do |config|
      
        # Configuração da Box
        config.vm.box = "debian/bookworm64"
      
        # Configuração da VM
        config.vm.provider "virtualbox" do |vb|
          vb.name = "p01_isabel" # Nome da VM
          vb.memory = 1024       # Memória RAM
      
          # Adicionando discos adicionais
          vb.customize ["createhd", "--filename", "disk1.vdi", "--size", 10240]
          vb.customize ["createhd", "--filename", "disk2.vdi", "--size", 10240]
          vb.customize ["createhd", "--filename", "disk3.vdi", "--size", 10240]
          vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 1, "--device", 0, "--type", "hdd", "--medium", "disk1.vdi"]
          vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 2, "--device", 0, "--type", "hdd", "--medium", "disk2.vdi"]
          vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", 3, "--device", 0, "--type", "hdd", "--medium", "disk3.vdi"]
      
        end
      
        # Configuração de rede
        config.vm.network "private_network", ip: "192.168.57.10"
      
        # Configuração de provisionamento com Ansible
        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "playbook.yml"
        end
      end
   
#
## Playbook
O Playbook Ansible define a execução das tarefas, as mesmas são organizadas em roles provisionando e configurando o sistema.

        - hosts: all
        become: true
        roles:
          - atualizar_sistema
          - configurar_hostname
          - criar_usuarios
          - mensagem_saudacao
          - configurar_sudo
          - configurar_ssh
          - configurar_lvm
          - configurar_nfs
          - monitorar_acessos

As roles, são um agrupamento de tarefas que podem ser reutilizadas para a configuração de servidores ou de sistemas. No projeto foi solicitado os tipos de tarefas a serem desenvolvidas. 

### **Atualizar Sistema** Atualiza os pacotes e o sistema operacional.
       
         - name: Atualizar Sistema Operacional  
        apt:  
          update_cache: yes  
          upgrade: dist
       
### **Configurar Hostname** 
Define o nome do host para o sistema.

      - name: Configurar Hostname
         hostname:
           name: "p01-samuel-isabel"

### **Criar Usuários** 
O objetivo é configurar rapidamente um ambiente com grupos e usuários, facilitando o controle de acesso, especialmente para conexões via SSH.

      - name: Criar grupo acesso_ssh
        ansible.builtin.group:
          name: acesso_ssh
          state: present
      
      - name: Criar grupo "ifpb"
        ansible.builtin.group:
          name: ifpb
          state: present
          
      - name: Criar Usuários
        ansible.builtin.user:
          name: "{{ item }}"
          groups: "ifpb,acesso_ssh"
          state: present
        loop:
         - samuel
         - isabel


### **Mensagem de Saudação:** 
Configura uma mensagem de boas-vindas (motd) exibida no terminal ao fazer login.

      - name: Configurar Mensagem de Saudação  
        copy:  
          content: |  
            Acesso restrito apenas a pessoas com autorização expressa.  
            Seu acesso está sendo monitorado !!!  
          dest: /etc/motd

### **Configurar Sudo** 
O código permite que os membros do grupo ifpb tenham acesso sudo sem exigir senha, criando um arquivo de configuração apropriado em /etc/sudoers.d/.
      
      - name: Permitir que o grupo "ifpb" tenha acesso SUDO  
        copy:  
          dest: /etc/sudoers.d/ifpb  
          content: "%ifpb ALL=(ALL) NOPASSWD:ALL"  
          mode: "0440"

### **Configurar SSH** 

o playbook a seguir tem o objetivo de fazer uma configuração ssh de forma mais segura, o código tem  composição de tarefas que
fazem a configuração SSH, ajustando  suas  necessidades de segurança e garantido que apenas usuários autorizados possam estar fazendo acesso.

Primeiramente, o playbook desabilita a autenticação por senha, configurando o arquivo de configuração do SSH ( /etc/ssh/sshd_config) **PasswordAuthentication** para **no**

Após realizar essas mudanças, o playbook reinicia o serviço SSH para garantir que as novas configurações entrem em vigor. Esse reinício é essencial para que o servidor comece a aplicar as restrições e políticas de segurança.

Uma parte importante de todo este processo, é a criação do diretório **~/.ssh** para os usuários **Samuel** e **isabel**, **0700**). As chaves públicas desses usuários são copiadas para os arquivos **~/.ssh/authorized_keys**, permitindo o acesso via chave pública.

Por fim, é feita a limitação para acesso apenas ao grupo **acesso_ssh** garantido assim que apenas os membros possam estar fazendo o acesso SSH. aumentando a segurança do sistema, permitindo apenas acessos autorizados.


      # Configurar o serviço SSH
      - name: Permitir apenas chaves públicas
        lineinfile:
          path: /etc/ssh/sshd_config
          state: present
          regexp: "^PasswordAuthentication"
          line: "PasswordAuthentication no"
      
      - name: Bloquear root via SSH
        lineinfile:
          path: /etc/ssh/sshd_config
          state: present
          regexp: "^PermitRootLogin"
          line: "PermitRootLogin no"
      
      - name: Reiniciar serviço SSH
        service:
          name: ssh
          state: restarted
      
      # Configurar chaves públicas para os usuários
      - name: Garantir que o diretório ~/.ssh existe para todos os usuários
        file:
          path: "/home/{{ item }}/.ssh"
          state: directory
          mode: '0700'
          owner: "{{ item }}"
          group: "{{ item }}"
        with_items:
          - samuel
          - isabel
          
      - name: Copiar chave pública de samuel para os usuários remotos
        authorized_key:
          user: "{{ item }}"
          state: present
          key: "{{ lookup('file', '~/Downloads/projeto/samuel_rsa.pub') }}"
        with_items:
          - samuel
          
      - name: Copiar chave pública de isabel para os usuários remotos
        authorized_key:
          user: "{{ item }}"
          state: present
          key: "{{ lookup('file', '~/Downloads/projeto/isabel_rsa.pub') }}"
        with_items:
          - isabel
          
      - name: Permitir acesso apenas ao grupo "acesso_ssh"
        lineinfile:
          path: /etc/ssh/sshd_config
          regexp: "^#?AllowGroups"
          line: "AllowGroups acesso_ssh"


### **Configurar LVM** 
A tarefa realiza a instalação dos pacotes necessários para utilizar a funcionalidade LVM (Logical Volume Management). Em seguida, cria um grupo de volumes chamado "dados" e um volume lógico chamado "sistema", com o tamanho especificado. Após a criação do volume lógico, ele é formatado no sistema de arquivos ext4. Finalmente, um diretório é criado para que o volume lógico seja montado corretamente no sistema.

---
      - name: Instalar pacotes necessários
        apt:
          name: "{{ item }}"
          state: present
        loop:
          - lvm2
      
      - name: Criar Volume Group chamado "dados"
        lvg:
          vg: dados
          pvs:
            - /dev/sdb
            - /dev/sdc
            - /dev/sdd
      
      - name: Criar Logical Volume "sistema"
        community.general.lvol:
          vg: dados
          lv: sistema
          size: 15g
      
      - name: Formatar o LV "sistema" como ext4
        filesystem:
          fstype: ext4
          dev: /dev/dados/sistema
      
      - name: Criar diretório /dados
        file:
          path: /dados
          state: directory
      
      - name: Montar o LV "sistema" em /dados
        mount:
          path: /dados
          src: /dev/dados/sistema
          fstype: ext4
          state: mounted
      
      - name: Configurar montagem automática no /etc/fstab
        mount:
          path: /dados
          src: /dev/dados/sistema
          fstype: ext4
          opts: defaults
          state: present
          
### **Configurar NFS** 
Configura o serviço NFS para compartilhamento de arquivos entre sistemas.
Esta tarefa realiza a instalação dos pacotes necessários para o servidor NFS, cria um usuário específico que não pode realizar login diretamente, configura o diretório compartilhado pelo NFS, adiciona as permissões adequadas no arquivo /etc/exports para permitir o acesso à rede local e reinicia o serviço do servidor NFS após as configurações.

      - name: Instalar o servidor NFS
        apt:
          name: nfs-kernel-server
          state: present
      
      - name: Criar usuário "nfs-ifpb" para NFS
        user:
          name: nfs-ifpb
          shell: /usr/sbin/nologin
      
      - name: Criar diretório NFS
        file:
          path: /dados/nfs
          state: directory
          owner: nfs-ifpb
          group: nfs-ifpb
      
      - name: Dar permissão 777 ao diretório NFS
        file:
          path: /dados/nfs
          mode: "0777"
          owner: nfs-ifpb
          group: nfs-ifpb
      
      - name: Configurar diretório compartilhado NFS
        blockinfile:
          path: /etc/exports
          block: |
            /dados/nfs 192.168.57.0/24(rw,sync,no_subtree_check,all_squash,anonuid=1001,anongid=1001)
      
      - name: Reiniciar o servidor NFS
        service:
          name: nfs-kernel-server
          state: restarted

### **Monitorar acessos** 

      - name: Criar script de monitoramento
        copy:
          content: |
            #!/bin/bash
            echo "$(date '+%Y-%m-%d %H:%M'); $USER; $(tty); $SSH_CONNECTION" >> /dados/nfs/acessos
          dest: /usr/local/bin/monitor.sh
          mode: 0755
      
      - name: Configurar script para ser executado no login
        lineinfile:
          path: /etc/profile
          line: "/usr/local/bin/monitor.sh"
          state: present

O conteúdo descreve a tarefa, que consiste em configurar um script de monitoramento de acessos. O módulo copy é utilizado para criar ou modificar arquivos no destino especificado. A diretiva dest define o caminho onde o script será salvo, que é /etc/profile.d/monitor_acesso.sh. 
#
# 5.Desafios na Produção do Projeto de Infraestrutura com Vagrant e Ansible
### Configuração de Rede e Acesso SSH

A configuração da rede privada e o acesso via SSH apresentaram desafios, especialmente no que diz respeito à segurança. O bloqueio do acesso root e a desativação da autenticação por senha exigiram testes rigorosos para garantir que o acesso por chaves públicas estivesse funcionando corretamente. Problemas com permissões de diretórios e configurações inconsistentes do SSH dificultaram o progresso em alguns momentos.

### Configuração do NFS
A definição das permissões corretas para os diretórios compartilhados foi crucial, uma vez que permissões inadequadas impediam a escrita de arquivos por determinados usuários.

# 6.Testes e Funcionalidade
### Verificação da conectividade.

      samuel@samuel-Latitude-5430:~/Downloads/projeto$ ssh -i ~/Downloads/projeto/isabel_rsa isabel@192.168.57.10  
      The authenticity of host '192.168.57.10 (192.168.57.10)' can't be established.  
      ED25519 key fingerprint is SHA256:
#
### NFS
   
      samuel@samuel-Latitude-5430:~/Downloads/projeto$ sudo mkdir /mnt/nfs  
      samuel@samuel-Latitude-5430:~/Downloads/projeto$ ls  
      disk1.vdi  disk3.vdi  isabel_rsa.pub  roles  samuel_rsa  playbook.yml  Vagrantfile  
      samuel@samuel-Latitude-5430:~/Downloads/projeto$ sudo mount 192.168.57.10:/dados/nfs /mnt/nfs  
      samuel@samuel-Latitude-5430:~/Downloads/projeto$ df -h /mnt/nfs  
      Sist. Arq.  Tam.  Usado  Disp.  Uso% Montado em  
      192.168.57.10:/dados/nfs  15G   0   14G   0%  /mnt/nfs  
      samuel@samuel-Latitude-5430:~/Downloads/projeto$

 A pasta /dados/nfs está sendo compartilhada para a rede 192.168.57.0/24 com configurações para otimizar desempenho e segurança. As operações de escrita são síncronas (sync) e agrupadas (wdelay), enquanto verificações de subárvore são desativadas (hide, no_subtree_check) para melhorar a performance. O acesso anônimo é controlado com **anonuid=1001** e **anongid=1001**. O compartilhamento permite leitura e escrita (rw), utiliza segurança básica (sec=sys) e restringe acessos com secure. A opção **no_root_squash** garante que o usuário root no cliente tenha acesso root no servidor.
#         
### LVM

      $ sudo blkid  
      sudo: unable to resolve host p01-Isabel: Temporary failure in name resolution  
      /dev/sdb: UUID="S3ME08-MTej-CCXX-11yG-tId-0dp6-93uphI" TYPE="LVM2_member"  
      /dev/sdb1: UUID="V5rI51-Br9b-dgAk-NZnq-zYKj-FpVj-GJdtRx" TYPE="LVM2_member"  
      /dev/mapper/dados-sistema: UUID="8b7b1fa9-3757-4ae8-a6d4-314e8c458124" BLOCK_SIZE="4096" TYPE="ext4"  
      /dev/sdc: UUID="rcFaXt-6NSv-qcBd-AyB4-Tp60-Plf-OJdQit" TYPE="LVM2_member"  
      /dev/sda1: UUID="5b9b976f-334e-4d64-89e9-5309f9c88998" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="1923d3e-01"  
      PE="ext4"  
      
      $ df -h  
      Filesystem      Size  Used Avail Use% Mounted on  
      udev            459M     0  459M   0% /dev  
      tmpfs           97M   536K   96M   1% /run  
      /dev/sda1       98G  1.7G   92G   2% /  
      tmpfs           481M     0  481M   0% /dev/shm  
      tmpfs           5.0M     0  5.0M   0% /run/lock  
      tmpfs           97M     0   97M   0% /run/user/1000  
      vagrant        206G   30G  177G  15% /vagrant  
      /dev/mapper/dados-sistema  15G   24K   14G   1% /dados  
      tmpfs           97M     0   97M   0% /run/user/1001  
      
      $ cat /etc/fstab  
      # /etc/fstab: static file system information.  
      # <file sys>    <mount point>  <type>  <options>         <dump>  <pass>  
      # device during installation: /dev/loop0p1  
      UUID=5b9b976f-334e-4d64-89e9-5309f9c88998 / ext4 rw,discard,errors=remount-ro 0 0  
      #VAGRANT-BEGIN  
      # The contents below are automatically generated by Vagrant. Do not modify.  
      vagrant  /vagrant vboxsf uid=1000,gid=1000,_netdev 0 0  
      #VAGRANT-END  
      /dev/dados/sistema /dados ext4 defaults 0 0  
      $
#
### SSH PASSWORD E ROOT 

      $ sudo cat /etc/ssh/sshd_config | grep PasswordAuthentication  
      sudo: unable to resolve host p01-Isabel: Temporary failure in name resolution  
      PasswordAuthentication no  
      # PasswordAuthentication. Depending on your PAM configuration,  
      # PAM authentication, then enable this but set PasswordAuthentication  
      no  
      
      $ sudo cat /etc/ssh/sshd_config | grep PermitRootLogin  
      sudo: unable to resolve host p01-Isabel: Temporary failure in name resolution  
      #PermitRootLogin prohibit-password  
      PermitRootLogin no
#
# 7. Comandos para testes.
### Comando: 
      ssh -i ~/Downloads/projeto/isabel_rsa isab@192.168.57.10
   
Conexão via SSH a um servidor com o IP 192.168.57.10, usando a chave de autenticação localizada em ~/Downloads/projeto/isabel_rsa.
#
### Comandos:
      ls -l /usr/local/bin/monitor.sh
      cat /usr/local/bin/monitor.sh
 O primeiro comando lista detalhes sobre o arquivo monitor.sh, mostrando que ele é um script executável e que pertence ao usuário root. O segundo comando exibe o conteúdo do script, que gera um log com a data, usuário, terminal e conexão SSH. Isso indica que o acesso aos usuários está sendo monitorado.
 #
### Comandos:
      cat /dados/nfs/acessos
      sudo vgdisplay dados
      sudo lvdisplay /dev/dados/sistema
   O primeiro comando exibe o log de acessos gerado pelo script. Os seguintes comandos exibem informações sobre Volume Group (VG) e Logical Volume (LV), mostrando as propriedades e a utilização do espaço em disco.
#
### Comando: 
      ls -ld /dados/nfs
   Mostra a listagem do diretório /dados/nfs, indicando suas permissões e detalhes de propriedade. Este diretório é relevante para o armazenamento de dados NFS.
   #
### Comandos:
      sudo mkdir /mnt/nfs
      sudo mount 192.168.57.10:/dados/nfs /mnt/nfs
      df -h /mnt/nfs
   O primeiro comando cria um diretório para a montagem do NFS. O segundo comando monta o diretório do servidor NFS no diretório local criado. O terceiro comando verifica o uso do sistema de arquivos montado

  # 8.Conclusão
   A demais, foram realizados testes para verificar as outras fazes, os mesmos foram agrupados em Dados do projeto Teste https://github.com/Bellsatu/Vagrant/tree/4b275c0e2203c87fe6c095a2bf9d45b3e01e3bb4/Dados%20do%20projeto%20Testes. Ao fazer os testes no NFS, foi observado que ao tentar escrever um arquivo na pasta remota, o usuário deve atender a requisitos de permissão específicos.
O arquivo pode ser colocado na pasta remota apenas se o usuário tiver permissões de 777.
   Se o usuário tiver permissões de 775, ele não conseguirá realizar a operação de escrita.
Conforme mencionado, após ajustar as permissões para 777 para o usuário Samuel, a operação foi bem-sucedida.
   As permissões são cruciais para a operação de escrita na pasta remota. É necessário garantir que os usuários tenham as permissões corretas para evitar falhas na transferência de arquivos.
   Entre outras conjunturas foi verificado o acesso via SSH utilizando chaves públicas, e confirmando que o acesso por senha e o login com root estão bloqueados. Verificado que o volume LVM foi criado e montado corretamente, e o compartilhamento NFS estava funcionando com as permissões esperadas, levantando apenas as resalvas supracitadas. O monitoramento de acessos também funcionou, registrando corretamente os logins no arquivo de log.
      
      
