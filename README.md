Installation
------------ 
 - Install Virualbox https://virtualbox.org and Vagrant http://vagrantup.com
 - run provision script
```
vagrant up clickhouse_zetcd --provision
```

 - connect to virtualbox and run check scripts
 
```
vagrant ssh clickhouse_zetcd 
sudo bash
cd /vagrant
docker-compose up -d
docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "CREATE DATABASE IF NOT EXISTS zetcd_test"
docker-compose run clickhouse-client.local -h clickhouse-ru-2.local --echo -q "CREATE DATABASE IF NOT EXISTS zetcd_test"
docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "CREATE TABLE IF NOT EXISTS zetcd_test.test_replicated (timestamp UInt32, date MATERIALIZED toDate(timestamp), trackerId String,   userId String  ) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/hits_replicated', '{replica}', date, cityHash64( userId), (trackerId, date, cityHash64(userId), timestamp), 8192);"
docker-compose run clickhouse-client.local -h clickhouse-ru-2.local --echo -q "CREATE TABLE IF NOT EXISTS zetcd_test.test_replicated (timestamp UInt32, date MATERIALIZED toDate(timestamp), trackerId String,   userId String  ) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/hits_replicated', '{replica}', date, cityHash64( userId), (trackerId, date, cityHash64(userId), timestamp), 8192);"
docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "INSERT INTO zetcd_test.test_replicated (timestamp, trackerId, userId ) VALUES (1, 'test1','test1')"
docker-compose run clickhouse-client.local -h clickhouse-ru-2.local --echo -q "INSERT INTO zetcd_test.test_replicated (timestamp, trackerId, userId ) VALUES (1, 'test2','test2')"
docker-compose run clickhouse-client.local -h clickhouse-ru-1.local --echo -q "SELECT * FROM zetcd_test.test_replicated"
docker-compose run clickhouse-client.local -h clickhouse-ru-2.local --echo -q "SELECT * FROM zetcd_test.test_replicated"

# search something like <Debug> zetcd_test.test_replicated (StorageReplicatedMergeTree, RestartingThread): Activating replica.
docker-compose logs clickhouse-ru-1.local | less
docker-compose logs zktraffic
```
