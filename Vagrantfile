# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty32"
  config.vm.network "forwarded_port", guest: 3001, host: 3001  # 3001 is default insights api port
  config.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, "--hwvirtex", "off"]
      v.customize ["modifyvm", :id, "--vtxvpid", "off"]
      v.memory = 4096  # Min suggested ram from https://github.com/bitpay/bitcore-node
      v.cpus = 2
  end

  $initial_setup = <<-SHELL
    #!/bin/bash

    # Install pre-reqs
    sudo apt-get update
    sudo apt-get install -y build-essential git curl libzmq3-dev

    # Setup nvm to install proper version of npm
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.32.1/install.sh | bash
    echo "source ~/.nvm/nvm.sh" >> ~/.profile
    source ~/.profile
    nvm install v8.2.0
    nvm use v8.2.0

    # Clone bitcore-node
    git clone https://github.com/bitpay/bitcore-node.git ~/bitcore-node
    cd ~/bitcore-node
    npm install

    # Setup bitcore-node in a folder called 'mynode', located in home directory
    cd ~
    ~/bitcore-node/bin/bitcore-node create mynode

    # Setup insight-api on the bitcore-node
    cd ~/mynode/node_modules
    git clone https://github.com/bitpay/insight-api.git
    cd insight-api
    npm install

    # Insert insight-api service into bitcore-node.json config file
    cd ~/mynode
    sed -i '/\"web\"/ a ,\\n\"insight-api\"' bitcore-node.json
  SHELL
  config.vm.provision "shell", run: "once", privileged: false, inline: $initial_setup


  # Starts up bitcore-node and Insight API
  $system_startup = <<-SHELL
    # Launch a live instance of insight-api and bitcore-node
    screen -S mynode -mL -d bash -c 'cd ~/mynode;~/bitcore-node/bin/bitcore-node start'
  SHELL
  config.vm.provision "shell", run: "always", privileged: false, inline: $system_startup
end
