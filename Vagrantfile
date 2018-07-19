# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  config.vm.network "forwarded_port", guest: 4443, host: 4443
  config.vm.network "forwarded_port", guest: 443, host: 1443

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 443, host: 443, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
    vb.memory = "4096"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  
  config.vm.provision "file", source: "./configurations", destination: "/home/vagrant/configurations"

  config.vm.provision "shell", inline: <<-SHELL
   
    # update hosts file to work with made up urls
    sudo echo "127.0.0.1 interview.dds.com" >> /etc/hosts
    sudo echo "127.0.0.1 pushresults.com" >> /etc/hosts

    # grab software
    sudo apt-get update
    sudo apt-get install git -y
    sudo apt-get install apache2 -y

    # generate self signed ssl credentials for apache
    openssl req -x509 -newkey rsa:4096 -keyout apache.key -out apache.crt -days 365 -subj "/C=US/ST=Georgia/L=Augusta/O=DDS/OU=Abe/CN=pushresults.com" -nodes
    sudo cp /home/vagrant/apache.key /etc/ssl/private/
    sudo cp /home/vagrant/apache.crt /etc/ssl/certs/

    # configure https on apache
    sudo cp /home/vagrant/configurations/ssl-params.conf /etc/apache2/conf-available/
    sudo rm /etc/apache2/sites-available/default-ssl.conf
    sudo cp /home/vagrant/configurations/default-ssl.conf /etc/apache2/sites-available/
    sudo rm /etc/apache2/sites-available/000-default.conf
    sudo cp /home/vagrant/configurations/000-default.conf /etc/apache2/sites-available/
    sudo a2enmod ssl
    sudo a2enmod headers
    sudo a2ensite default-ssl
    sudo a2enconf ssl-params
    sudo service apache2 restart

    # add gitlab repo
    curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
    
    # install gitlab community edition
    sudo EXTERNAL_URL="https://interview.dds.com:4443" GITLAB_ROOT_PASSWORD="empiredidnothingwrong" apt-get install gitlab-ce
    
    # generate self signed ssl credentials for gitlab
    openssl req -x509 -newkey rsa:4096 -keyout interview.dds.com.key -out interview.dds.com.crt -days 365 -subj "/C=US/ST=Georgia/L=Augusta/O=DDS/OU=Abe/CN=interview.dds.com" -nodes
    sudo mkdir -p /etc/gitlab/ssl
    sudo chmod 700 /etc/gitlab/ssl
    sudo cp /home/vagrant/interview.dds.com.key /home/vagrant/interview.dds.com.crt /etc/gitlab/ssl/
    
    # reconfigure gitlab to utilize ssl credentials
    sudo echo "nginx['ssl_certificate'] = '/etc/gitlab/ssl/interview.dds.com.crt'" >> /etc/gitlab/gitlab.rb
    sudo echo "nginx['ssl_certificate_key'] = '/etc/gitlab/ssl/interview.dds.com.key'" >> /etc/gitlab/gitlab.rb
    sudo gitlab-ctl reconfigure

    # create a personal access token for interacting with api
    sudo gitlab-rails r "token = PersonalAccessToken.new(user: User.where(id: 1).first, name: 'api token', token: 'empiredidnothingwrong', scopes: ['api','read_user','sudo']); token.save"'!' 
    
    # create a ssh key
    sudo ssh-keygen -t rsa -C "admin@example.com" -b 4096 -N 'empiredidnothingwrong' -f /home/vagrant/.ssh/id_rsa
    sudo chown vagrant /home/vagrant/.ssh/id_rsa
    sudo chgrp vagrant /home/vagrant/.ssh/id_rsa
    curl -k -X POST --header "Private-Token: empiredidnothingwrong" https://localhost:4443/api/v4/user/keys --data-urlencode "title=$(hostname)" --data-urlencode "key=$(cat /home/vagrant/.ssh/id_rsa.pub)"

    # rename the root account to admin1
    curl -k -X PUT --header "Private-Token: empiredidnothingwrong" https://localhost:4443/api/v4/users/1\?username\=admin1
    curl -k -X PUT --header "Private-Token: empiredidnothingwrong" https://localhost:4443/api/v4/users/1\?name\=admin

    # create a new project named admin
    curl -k -X POST --header "Private-Token: empiredidnothingwrong" https://localhost:4443/api/v4/projects\?name\=admin

    # configure git settings
    git config --global user.name "admin1"
    git config --global user.email "admin@example.com"
    git config --global http.sslCAinfo /home/vagrant/interview.dds.com.crt
    git config --global push.default simple
    touch /home/vagrant/.gitconfig
    echo "[user]" >> /home/vagrant/.gitconfig
    echo "  name = admin1" >> /home/vagrant/.gitconfig
    echo "  email = admin@example.com" >> /home/vagrant/.gitconfig
    echo "[http]" >> /home/vagrant/.gitconfig
    echo "  sslCAinfo = /home/vagrant/interview.dds.com.crt" >> /home/vagrant/.gitconfig
    echo "[push]" >> /home/vagrant/.gitconfig
    echo "  default = simple" >> /home/vagrant/.gitconfig

    # clone the admin repository
    git clone https://admin1:empiredidnothingwrong@interview.dds.com:4443/admin1/admin.git
    sudo chown -R vagrant:vagrant /home/vagrant/admin

    # add git to sudoers
    sudo chmod +w /etc/sudoers
    sudo echo "git  ALL=NOPASSWD: ALL" >> /etc/sudoers
    sudo chmod -w /etc/sudoers

    # create a custom git hook for running scripts and publishing it on the web server
    sudo mkdir /var/opt/gitlab/git-data/repositories/admin1/admin.git/custom_hooks
    cd /var/opt/gitlab/git-data/repositories/admin1/admin.git/custom_hooks
    sudo touch post-receive
    sudo chmod +x post-receive
    sudo chown git post-receive
    sudo chgrp git post-receive
    sudo echo "#!/bin/bash" >> post-receive
    sudo echo "sudo rm /var/www/html/index.html" >> post-receive
    sudo echo "sudo touch /var/www/html/index.html" >> post-receive
    sudo echo "sudo chmod ugo+w /var/www/html/index.html" >> post-receive
    sudo echo "cd /etc" >> post-receive
    sudo echo "sudo git clone -c http.sslcainfo=/home/vagrant/interview.dds.com.crt https://admin1:empiredidnothingwrong@interview.dds.com:4443/admin1/admin.git" >> post-receive
    sudo echo "cd /etc/admin" >> post-receive
    sudo echo "for script in /etc/admin/*.sh; do" >> post-receive
    sudo echo '     sudo chmod +x $script' >> post-receive
    sudo echo '     sudo $script >> /var/www/html/index.html' >> post-receive
    sudo echo "done" >> post-receive
    sudo echo "cd /etc" >> post-receive
    sudo echo "sudo rm -rf admin" >> post-receive
    sudo echo "sudo chmod ugo-w /var/www/html/index.html" >> post-receive

  SHELL
end
