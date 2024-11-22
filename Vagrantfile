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

      # Crear directorio para el proyecto
      mkdir -p /var/www/pedro/html
      git clone https://github.com/cloudacademy/static-website-example /var/www/pedro/html
      chown -R www-data:www-data /var/www/pedro/html
      chmod -R 755 /var/www/pedro

      # Configuración para el sitio web
      mkdir -p /var/www/perfectweb/html
      cp -r /vagrant/html/* /var/www/perfectweb/html/
      chown -R www-data:www-data /var/www/perfectweb/html
      chmod -R 755 /var/www/perfectweb

      # Autenticación
      sh -c "echo 'pedro:$(openssl passwd -apr1 1234)' >> /etc/nginx/.htpasswd" 
      sh -c "echo 'gallegos:$(openssl passwd -apr1 1234)' >> /etc/nginx/.htpasswd"

      # Configuración pedro
      cp -v /vagrant/pedro /etc/nginx/sites-available/pedro
      ln -s /etc/nginx/sites-available/pedro /etc/nginx/sites-enabled/

      # Confuguración perfectweb
      cp -v /vagrant/perfectweb /etc/nginx/sites-available/perfectweb
      ln -s /etc/nginx/sites-available/perfectweb /etc/nginx/sites-enabled/

      # Verificar servicios
      nginx -t
      
      # Reiniciar servicios
      systemctl restart nginx
    SHELL
  end # pedro
end
