# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.box_check_update = false

  config.vm.define :clickhouse_zetcd do |clickhouse_zetcd|
        clickhouse_zetcd.vm.network "private_network", ip: "172.16.1.77"
        clickhouse_zetcd.vm.host_name = "local-zetcd-clickhouse-pro"
  end

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = false
  
    # Customize the amount of memory on the VM:
    vb.memory = "2094"
  end
  config.vm.provision "shell", inline: <<-SHELL
    sudo sysctl net.ipv6.conf.all.forwarding=1 && \
    sudo apt-get update && \
    sudo apt-get install -y apt-transport-https software-properties-common aptitude && \
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E0C56BD4 && \
	sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 58118E89F3A912897C070ADBF76221572C52609D && \
    sudo add-apt-repository "deb http://repo.yandex.ru/clickhouse/xenial/ stable main" && \
    sudo add-apt-repository "deb https://apt.dockerproject.org/repo ubuntu-xenial main" && \
    sudo apt-get update && \
    sudo apt-get install -y docker-engine && \
    sudo apt-get install -y clickhouse-client && \
    sudo apt-get install -y python-pip && \
    sudo apt-get install -y htop ethtool mc && \
    sudo python -m pip install -U pip && \
    sudo pip install -U docker-compose && \
    cd /vagrant && \
  	sudo docker-compose up -d && \
	sudo docker-compose run clickhouse-client.local -h clickhouse-ru-1.local -q "CREATE DATABASE IF NOT EXISTS zetcd_test" && \
	sudo docker-compose run clickhouse-client.local -h clickhouse-ru-2.local -q "CREATE DATABASE IF NOT EXISTS zetcd_test" && \
	sudo docker-compose run clickhouse-client.local -h clickhouse-ru-1.local -q "CREATE TABLE IF NOT EXISTS zetcd_test.test_replicated (timestamp UInt32, date MATERIALIZED toDate(timestamp), trackerId String,   userId String  ) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/hits_replicated', '{replica}', date, cityHash64( userId), (trackerId, date, cityHash64(userId), timestamp), 8192);" && \
	sudo docker-compose run clickhouse-client.local -h clickhouse-ru-2.local -q "CREATE TABLE IF NOT EXISTS zetcd_test.test_replicated (timestamp UInt32, date MATERIALIZED toDate(timestamp), trackerId String,   userId String  ) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/hits_replicated', '{replica}', date, cityHash64( userId), (trackerId, date, cityHash64(userId), timestamp), 8192);" && \
	sudo docker-compose logs zktraffic
	echo "click PROVISIONING DONE, Good Luck ;)"
  SHELL
end
