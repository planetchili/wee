Vagrant.configure("2") do |config|

  # Check for missing plugins and install as necessary
  #
  required_plugins = ["vagrant-vbguest"]
  plugin_installed = false
  required_plugins.each do |plugin|
    unless Vagrant.has_plugin?(plugin)
      system "vagrant plugin install #{plugin}"
      plugin_installed = true
    end
  end  
  # If new plugins installed, restart Vagrant process
  if plugin_installed === true
    exec "vagrant #{ARGV.join' '}"
  end


  # specify base vagrant box
  #
  config.vm.box = "bento/centos-7.4"

  
  # Disable automatic box update checking.
  #
  config.vm.box_check_update = false
  
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  #
  config.vm.network "private_network", ip: "192.168.33.20"

  
  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  #
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "wexer", "/var/www/wexer"

  
  # Provider-specific configuration 
  #
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
  end
    

  # Provisioning with a shell script.
  #
  config.vm.provision "shell", inline: <<-SHELL
  
	################ yum repo setup #############
    yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
	yum -y install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
	yum -y install yum-utils
	yum -y update
	
	############ MYSQL (MariaDB) ################
	# install
	yum -y install mariadb-server mariadb
	# start service and enable autostart
	systemctl start mariadb.service
	systemctl enable mariadb.service
	# setup root user
	mysql -u root -e "DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost')"
	mysql -u root -e "CREATE USER 'root'@'%' IDENTIFIED BY ''"
	mysql -u root -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION"
	# remove anonymous user and test db
	mysql -u root -e "DELETE FROM mysql.db WHERE db LIKE 'tes%' AND user=''"
	mysql -u root -e "DELETE FROM mysql.user WHERE User=''"
	mysql -u root -e "DROP DATABASE test"
	mysql -u root -e "FLUSH PRIVILEGES"
	# add site user and schema
	mysql -u root -e "CREATE DATABASE wexer"
	mysql -u root -e "CREATE USER 'wexer'@'localhost' IDENTIFIED BY ''"
	mysql -u root -e "GRANT ALL PRIVILEGES ON wexer.* TO 'wexer'@'localhost'"	
	
	############ Apache (HTTPD) ################
	# install
	yum -y install httpd
	# copy configuration	
	cp /var/www/wexer/.provision/httpd.conf /etc/httpd/conf
	
	############ PHP 7.2 ################
	# prioritize remi packages	
	yum-config-manager --enable remi-php72
	# install core
	yum -y install php php-cli php-opcache php-mysqlnd php-pdo
	# install auxilliary
	yum -y install php-gd php-mbstring php-json
	# install xdebug (and pear i guess too)
	yum -y install php-devel gcc gcc-c++ autoconf automake
	yum -y install php-pear
	pecl install Xdebug <<< ''
	# copy configuration
	cp /var/www/wexer/.provision/php.ini /etc
	
	################ Misc Utils ############
	# composer
	yum -y install composer
	
	########### Environment ################
	# symlink to sync (site) folder
	ln -s /var/www/wexer sync
	# remove root password
	passwd -d root
	
	## test shenanigans
	rmdir /var/www/html
	mkdir /var/www/wexer/public
	printf '<?php phpinfo();\n' > /var/www/wexer/public/index.php
	
  SHELL
  
  # start apache via script so it starts after site folder mounted
  #
  config.vm.provision :shell, :inline => "apachectl start", run: "always"  
  
end
