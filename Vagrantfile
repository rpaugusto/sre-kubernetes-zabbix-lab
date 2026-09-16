# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/rockylinux-9"
  config.vm.hostname = "sre-lab-k8s"

  # IP privado fixo conforme planejado no TXT
  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "sre-lab-k8s"
    vb.cpus = 2
    vb.memory = 4096 # Mantém folga nos 16GB de RAM do notebook
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--audio", "none"]
  end

  # CORREÇÃO DEFINITIVA: Roda tudo dentro da VM isolada
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "ansible/site.yml"
    ansible.install_mode = "pip" # Instala via pip para garantir compatibilidade no Rocky 9
    ansible.inventory_path = "ansible/inventory.ini"
    ansible.limit = "lab" 
  end
end
