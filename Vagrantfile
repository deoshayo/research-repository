# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Enable NFS access to the disk
  config.vm.synced_folder "..", "/vagrant", :nfs => true

  # Speed up DNS lookups
  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "off"]
  end

  # NFS requires a host-only network
  # This also allows you to test via other devices (e.g. mobiles) on the same
  # network
  config.vm.network :private_network, ip: "10.11.12.13"

  # Django dev server
  config.vm.network "forwarded_port", guest: 8000, host: 8000
  config.vm.network "forwarded_port", guest: 1080, host: 1080

  # Give the VM a bit more power to speed things up
  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 2
  end

  # Provision the vagrant box
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update

    cd /vagrant/repository

    #fix dpkg-preconfigure error
    export DEBIAN_FRONTEND=noninteractive
    
    # Install the packages from conf/packages
    xargs sudo apt-get install -qq -y < conf/packages
    # Install some of the other things we need that are just for dev
    sudo apt-get install -qq -y ruby-dev libsqlite3-dev build-essential

    # Create a postgresql user
    sudo -u postgres psql -c "CREATE USER repository SUPERUSER CREATEDB PASSWORD 'repository'"
    # Create a database
    sudo -u postgres psql -c "CREATE DATABASE repository"

    # Install mailcatcher to make dev email development easier
    sudo gem install --no-rdoc --no-ri --version "< 3" mime-types
    sudo gem install --no-rdoc --no-ri --conservative mailcatcher

    # Run post-deploy actions script to update the virtualenv, install the
    # python packages we need, migrate the db and generate the sass etc
    conf/post_deploy_actions.bash
  SHELL

  # Start mailcatcher every time we start the VM
  config.vm.provision "shell", run: "always" do |s|
    s.inline = <<-SHELL
      mailcatcher --http-ip 0.0.0.0
    SHELL
  end
end
