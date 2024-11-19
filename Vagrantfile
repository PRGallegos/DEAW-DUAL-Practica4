# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "base"

  config.vm.provision "shell", inline: <<-SHELL
    dpkg -l | grep openssl  
    apt-get update
    apt-get install -y apache2
  SHELL
end
