# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.define "pedro" do |pedro|
    # Configuración de la máquina virtual
    config.vm.box = "debian/bookworm64"
    config.vm.network "private_network", ip: "192.168.57.103"

    # Aprovisionamiento de la máquina
    config.vm.provision "shell", inline: <<-SHELL
      # Actualización del sistema e instalación de paquetes
      apt-get update -y
      apt-get install -y nginx openssl

      # Crear carpeta en la máquina
      mkdir -p /var/www/practica2.2/html

      # Copiar carpeta
      cp -r /vagrant/html /var/www/practica2.2/html


      # Configuración de permisos para la carpeta del sitio web
      chown -R www-data:www-data /var/www/practica2.2
      chmod -R 755 /var/www/practica2.2

      # Autenticación
      sh -c "echo 'pedro:$(openssl passwd -apr1 1234)' >> /etc/nginx/.htpasswd" 
      sh -c "echo 'gallegos:$(openssl passwd -apr1 1234)' >> /etc/nginx/.htpasswd"

      # Configuración servidor NGINX
      cp -v /vagrant/auth-basic.conf /etc/nginx/sites-available/pedro
      ln -s /etc/nginx/sites-available/pedro /etc/nginx/sites-enabled/

      # Verificar servicios
      systemctl status nginx

      # Reiniciar servicios
      systemctl restart nginx
    SHELL
  end # pedro
end
