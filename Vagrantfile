# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Usando Rocky Linux 9 oficial da Bento (Padrão Red Hat Enterprise)
  config.vm.box = "bento/rockylinux-9"
  config.vm.hostname = "sre-lab-k8s"

  config.vm.network "private_network", ip: "192.168.56.10"
  
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 4
    v.name = "sre-lab-k8s"
  end

  # Configuração para rodar o Ansible de DENTRO da VM Linux
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "ansible/site.yml"
    ansible.install_mode = "pip" # Instala via pip para garantir compatibilidade no Rocky 9
  end
end