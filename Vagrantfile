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
  config.vm.box = "centos/8"
  config.vm.box_version = "1905.1"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 3000, host: 3000
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1"

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
  config.vm.synced_folder ".", "/home/vagrant/project", type:"virtualbox"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
    # Customize the amount of memory on the VM:
    vb.memory = "4096"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    dnf update -y
    
    # nodejsのインストール、yarn
    dnf install -y nodejs
    npm -g install yarn

    # golangのインストール、GOPATH設定
    dnf install -y golang
    su - vagrant -c 'echo "export GOPATH=/home/vagrant/.go" >> .bash_profile'

    # gofishインストール、設定
    dnf install -y git # gofishの内部で使うため。
    su - vagrant -c "curl -fsSL https://raw.githubusercontent.com/fishworks/gofish/main/scripts/install.sh | bash"
    su - vagrant -c "gofish init"

    # buffaloのインストール
    su - vagrant -c "gofish install buffalo"

    # SELinux,firewalldの初期状態の確認
    echo SELinux status is ...
    getenforce
    echo firewalld status is ...
    systemctl enable firewalld
    systemctl start firewalld
    firewall-cmd --list-all

    # hostosからguestosの通信で指定のポートを開けておく。
    # firewall-cmd --add-port=3000/tcp --zone=public --permanent
    firewall-cmd --add-service=http --zone=public --permanent
    firewall-cmd --add-service=https --zone=public --permanent

    # nginxのインストール設定。
    dnf install -y nginx
    systemctl enable nginx

    # リバースプロキシの設定方法https://gobuffalo.io/en/docs/deploy/proxy
    # nginxのリバースプロキシを使う場合に必要なselinuxの設定
    setsebool -P httpd_can_network_connect on
    setsebool -P httpd_can_network_relay on

    # postgresqlのインストール,初期化
    dnf install -y postgresql-server postgresql-server-devel
    su - postgres -c "initdb -D /var/lib/pgsql/data -E UTF8 --no-locale"
    systemctl enable postgresql
    systemctl start postgresql
    su - postgres -c "psql -c 'CREATE DATABASE hello_buffalo_development'"

    # vagrantのホストOSとゲストOSのリアルタイムのファイル同期の機能を使うために必要。カーネルのビルドなので要再起動。
    # カーネルのビルド後にguest Aditionがインストールできるようになる
    # 再起動後にvagrant reloadによりGuest Aditionインストール。
    dnf install -y kernel-devel kernel-headers gcc gcc-c++
    reboot
  SHELL
end
