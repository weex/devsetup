# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "bstoots/xubuntu-16.04-desktop-amd64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

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
  config.vm.synced_folder "./data", "/home/vagrant/data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
      vb.memory = "2048"

      # in case cable isn't connected on start - observed on OS X 10.11.6 with
      # virtualbox 5.0.10
      vb.customize ['modifyvm', :id, '--cableconnected1', 'on']
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  RPCPASS = [*('A'..'Z')].sample(32).join

  config.vm.provision "shell", inline: <<-SHELL

      # network connectivity unreliable, add time for net to start
      sleep 30s

      add-apt-repository -y ppa:bitcoin/bitcoin
      apt-get update
      apt-get install -y terminator bitcoind virtualenv python3-pip sqlite3 libsodium-dev python-dev git-core
      apt-get remove python-pytest

      mkdir .bitcoin
      echo "rpcuser=bitcoinrpc" >> .bitcoin/bitcoin.conf
      echo "rpcpassword=#{RPCPASS}" >> .bitcoin/bitcoin.conf
      echo "testnet=1" >> .bitcoin/bitcoin.conf
      echo "server=1" >> .bitcoin/bitcoin.conf
      echo "prune=2000" >> .bitcoin/bitcoin.conf
      echo "walletnotify=curl -sI --connect-timeout 0.1 http://localhost:62612/walletnotify?%s" >> .bitcoin/bitcoin.conf
      echo "alertnotify=/usr/bin/wget -q --spider --timeout=0.5 --tries=1 http://localhost:62612/alertnotify?%s" >> .bitcoin/bitcoin.conf


      git clone https://github.com/ReinProject/python-rein.git
      virtualenv envpr
      . ./envpr/bin/activate
      cd python-rein
      pip install -r requirements.txt
      python setup.py install
      deactivate
      cd ..

      git clone https://github.com/ReinProject/causeway.git
      virtualenv --python=python3 envc
      . ./envc/bin/activate
      cd causeway
      pip3 install -r requirements.txt
      deactivate
      sqlite3 causeway.db < schema.sql
      echo "SERVER_PORT = 2016" >> settings.py
      echo "DATABASE_URI = 'sqlite:///home/vagrant/causeway/causeway.db'" >> settings.py
      echo "DEBUG = True" >> settings.py
      echo "TESTNET = True" >> settings.py
      echo "DATA_DIR = ''" >> settings.py
      echo "# Price in BTC for 1MB storage and 50MB transfer" >> settings.py
      echo "PRICE = 0.001" >> settings.py
      echo "" >> settings.py
      echo "# RPC to Bitcoin Core" >> settings.py
      echo "CORE_ENABLED = True" >> settings.py
      echo "SERVER = '127.0.0.1'" >> settings.py
      echo "RPCPORT = 18332" >> settings.py
      echo "RPCUSER = 'bitcoinrpc'" >> settings.py
      echo "RPCPASS = '#{RPCPASS}'" >> settings.py
      echo "" >> settings.py
      echo "# Minimum number of confirmations to consider a payment good" >> settings.py
      echo "MINCONF = 1" >> settings.py
      cd ..

      virtualenv envjm
      . ./envjm/bin/activate
      git clone https://github.com/JoinMarket-Org/joinmarket.git
      cd joinmarket
      git checkout develop
      cp test/regtest_joinmarket.cfg joinmarket.cfg
      sed -i -e "s/^rpc_user = .*/rpc_user = bitcoinrpc/" joinmarket.cfg
      sed -i -e "s/^rpc_password = .*/rpc_password = #{RPCPASS}/" joinmarket.cfg
      sed -i -e "s/^rpcuser=.*/rpcuser=bitcoinrpc/" test/bitcoin.conf
      sed -i -e "s/^rpcpassword=.*/rpc_password=#{RPCPASS}/" test/bitcoin.conf
      git clone https://github.com/JoinMarket-Org/miniircd.git
      pip install -r requirements.txt
      pip install pexpect pytest-cov mock
      
      echo ".PHONY: all test clean" >> Makefile
      echo "" >> Makefile
      echo "test:" >> Makefile
      echo -e "\tPYTHONPATH=.:/home/vagrant/envjm" >> Makefile
      echo -e "\tpy.test --cov-report html --btcroot=/usr/bin/ --btcconf=/home/vagrant/.bitcoin/bitcoin.conf --btcpwd=#{RPCPASS} --nirc=2 --ignore test/test_tumbler.py" >> Makefile
      deactivate
      cd ..
      # Steps to test joinmarket in regtest mode once this box is up:
      # comment out testnet=1 line in /home/vagrant/.bitcoin/bitcoin.conf
      # . envjm/bin/activate
      # cd joinmarket
      # make test

      sudo chown -R vagrant .bitcoin envpr python-rein envc causeway envjm joinmarket
  SHELL

end
