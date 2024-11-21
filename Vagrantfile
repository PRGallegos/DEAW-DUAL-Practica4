# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.define "pedro" do |pedro|
    # Configuración de la máquina virtual
    config.vm.box = "debian/bookworm64"
    config.vm.network "private_network", ip: "192.168.57.103"
    config.vm.hostname = "nginx-auth-practice"

    # Sincronización de carpetas
    config.vm.synced_folder "./files/nginx", "/vagrant/nginx-config"
    config.vm.synced_folder "./files/www", "/vagrant/www"

    # Aprovisionamiento de la máquina
    config.vm.provision "shell", inline: <<-SHELL
      # Actualización del sistema e instalación de paquetes
      apt-get update
      apt-get install -y nginx openssl

      # Configuración del directorio web
      mkdir -p /var/www/auth-practice
      cp -r /vagrant/www/* /var/www/auth-practice

      # Configuración de Nginx
      cp /vagrant/nginx-config/auth-practice /etc/nginx/sites-available/
      ln -s /etc/nginx/sites-available/auth-practice /etc/nginx/sites-enabled/
      rm -f /etc/nginx/sites-enabled/default

      # Archivo de autenticación
      cp /vagrant/nginx-config/htpasswd /etc/nginx/.htpasswd

      # Reinicio de Nginx para aplicar la configuración
      systemctl restart nginx
    SHELL
  end
end
