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
    vb.memory = "2048"
  end
  config.vm.provision "shell", inline: <<-SHELL
    set -xeuo pipefail
    sysctl net.ipv6.conf.all.forwarding=1
    apt-get update
    apt-get install -y apt-transport-https ca-certificates software-properties-common curl
    # clickhouse
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E0C56BD4
    # gophers
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 136221EE520DDFAF0A905689B9316A7BC7917B12
    # docker
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 8D81803C0EBFCD88
    add-apt-repository "deb http://repo.yandex.ru/clickhouse/deb/stable/ main/"
    add-apt-repository "deb https://download.docker.com/linux/ubuntu xenial edge"
    apt-get update
    apt-get install -y docker-ce
    apt-get install -y clickhouse-client
    apt-get install -y python-pip
    apt-get install -y htop ethtool mc
    python -m pip install -U pip
    pip install -U docker-compose
    cd /vagrant
    docker-compose down
    docker system prune -f
    docker-compose up -d
    docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "CREATE DATABASE IF NOT EXISTS zetcd_test"
    docker-compose run clickhouse-client.local -h clickhouse-ru-2.local --echo -q "CREATE DATABASE IF NOT EXISTS zetcd_test"
    docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "CREATE TABLE IF NOT EXISTS zetcd_test.test_replicated (timestamp UInt32, date MATERIALIZED toDate(timestamp), trackerId String,   userId String  ) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/hits_replicated', '{replica}', date, cityHash64( userId), (trackerId, date, cityHash64(userId), timestamp), 8192);"
    docker-compose run clickhouse-client.local -h clickhouse-ru-2.local --echo -q "CREATE TABLE IF NOT EXISTS zetcd_test.test_replicated (timestamp UInt32, date MATERIALIZED toDate(timestamp), trackerId String,   userId String  ) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/hits_replicated', '{replica}', date, cityHash64( userId), (trackerId, date, cityHash64(userId), timestamp), 8192);"
    docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "INSERT INTO zetcd_test.test_replicated (timestamp, trackerId, userId ) VALUES (1, 'test1','test1')"
    docker-compose run clickhouse-client.local -h clickhouse-ru-2.local --echo -q "INSERT INTO zetcd_test.test_replicated (timestamp, trackerId, userId ) VALUES (1, 'test2','test2')"
    docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "SELECT * FROM zetcd_test.test_replicated"
    docker-compose run clickhouse-client.local -h clickhouse-ru-2.local --echo -q "SELECT * FROM zetcd_test.test_replicated"
    # after second insert with same data will skipped
    docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "INSERT INTO zetcd_test.test_replicated (timestamp, trackerId, userId ) VALUES (1, 'test1','test1')"
    docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "INSERT INTO zetcd_test.test_replicated (timestamp, trackerId, userId ) VALUES (3, 'test3','test3')"
    docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "SELECT * FROM zetcd_test.test_replicated"
    docker-compose run clickhouse-client.local -h clickhouse-ru-2.local --echo -q "SELECT * FROM zetcd_test.test_replicated"

    # docker login -u clickhousepro
    # docker-compose build zetcd
    # docker-compose push zetcd
    # docker-compose logs zktraffic
    echo "clickhouse + zetcd PROVISIONING DONE, self-test PASSED, Good Luck ;)"
  SHELL
end
