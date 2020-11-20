# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define "client" do |client|
    client.vm.box = "bento/ubuntu-18.04"
    client.vm.hostname = "client"
    client.vm.network "private_network", type: "dhcp"
    client.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install casync
    SHELL
  end

  config.vm.define "server" do |server|
    server.vm.box = "bento/ubuntu-18.04"
    server.vm.hostname = "server"
    server.vm.network "private_network", type: "dhcp"
    server.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install casync
    SHELL
  end

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
  end

end
