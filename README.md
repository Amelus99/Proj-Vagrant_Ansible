# Projeto de Infraestrutura com Vagrant e Ansible

## Integrante
- Samuel

## Disciplina
Administração de Sistemas Abertos
Professor: Pedro Filho  

## Descrição do Projeto
Este projeto tem como objetivo desenvolver competências práticas em DevOps e Infraestrutura como Código (IaC), utilizando as ferramentas Vagrant e Ansible. Durante o projeto, foi provisionada uma máquina virtual e automatizada a configuração do sistema operacional e de serviços essenciais.

### Funcionalidades Implementadas
- **Atualização do sistema operacional**: Todos os pacotes foram atualizados.
- **Configuração do hostname**: O hostname foi alterado para `p01-samuel`.
- **Criação de usuários**: O usuário `samuel` foi criado com as permissões necessárias.
- **Mensagem de saudação (MOTD)**: Uma mensagem de boas-vindas foi configurada para exibição ao acessar via SSH.
- **Configuração de SUDO**: O grupo `ifpb` possui permissões de root sem necessidade de senha.
- **Configuração de SSH**:
  - Autenticação apenas por chaves públicas.
  - Bloqueio de acesso ao usuário root via SSH.
  - Acesso permitido apenas para usuários do grupo `acesso_ssh`.
- **Gerenciamento de discos com LVM**:
  - Três discos de 10 GB foram utilizados para criar um Volume Group (VG) chamado `dados`.
  - Um Logical Volume (LV) chamado `sistema` foi criado e formatado com o sistema de arquivos `ext4`.
  - O LV foi configurado para montagem automática no diretório `/dados`.
- **Configuração de NFS**:
  - O diretório `/dados/nfs` foi compartilhado com a rede `192.168.57.0/24`.
  - Configurado para que apenas o usuário `nfs-ifpb` tenha permissões de escrita.
  - Acesso de outros usuários remotos é mapeado para o usuário `nfs-ifpb`.
- **Monitoramento de acessos**:
  - Um script foi implementado para registrar data, usuário, dispositivo e IP remoto em um arquivo de log.

---

## Requisitos
Para executar este projeto, você precisa dos seguintes softwares instalados no host:
- **VirtualBox**
- **Vagrant**
- **Ansible**

---
