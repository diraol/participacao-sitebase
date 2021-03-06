# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$firstTimeScript = <<SCRIPT

cd /vagrant && composer update && rm -r /var/www/public && ln -s /vagrant/src /var/www/public

cp /vagrant/config/wp-config-vagrant.php /vagrant/src/wp-config.php
cp /vagrant/config/htaccess-vagrant /vagrant/src/.htaccess

service apache2 start

service mysql start

mysql --user=root --password=root -h 127.0.0.1 -e 'drop database participacao'
mysql --user=root --password=root -h 127.0.0.1 -e 'create database participacao'
cd /vagrant/db
bunzip2 db.sql.bz2
mysql --user=root --password=root -h 127.0.0.1 participacao < /vagrant/db/db.sql
bzip2 -9 db.sql

SCRIPT

$updateServices = <<SCRIPT

service mysql start

rm -r /var/www/public
ln -s /vagrant/src/ /var/www/public
chmod 777 /vagrant/src/wp-content/

service apache2 start

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    # Every Vagrant virtual environment requires a box to build off of.
    config.vm.box = "scotch/box"
    config.vm.network "forwarded_port", guest: 80, host: 80
    config.vm.network "forwarded_port", guest: 80, host: 8080
    # Port de debug do xdebug
    config.vm.network "forwarded_port", guest: 9000, host: 9000

    config.vm.provision "shell", inline: $firstTimeScript

    config.vm.provision "shell", inline: $updateServices,
            run: "always"

    config.trigger.after [:provision, :up, :reload] do
      system('echo "
        rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 80 -> 127.0.0.1 port 8080
        rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 443 -> 127.0.0.1 port 8443
        " | sudo pfctl -f - > /dev/null 2>&1; echo "==> Fowarding Ports: 80 -> 8080, 443 -> 8443"')
    end

    config.trigger.after [:halt, :destroy] do
        system("sudo pfctl -f /etc/pf.conf > /dev/null 2>&1; echo '==> Removing Port Forwarding'")
    end

    #config.vm.provider :virtualbox do |vb|
    #  vb.gui = true
    #end
end



