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
    vb.name = "p01_samuel_isabel" # Nome da VM
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
  config.vm.network "private_network", ip: "192.168.57.11"

  # Configuração de provisionamento com Ansible
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
end

