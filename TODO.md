= пакет clickhouse-test =

запустить clickhouse-test 
можно запустить только тесты связаные с zookeeper  -  clickhouse-test zookeeper
или оно же в dbms/tests/clickhouse-test

Ещё есть интеграционные тесты, чтоб они работали с zetcd нужно их немного поправить - тут: 
https://github.com/yandex/ClickHouse/blob/master/dbms/tests/integration/helpers/docker_compose_zookeeper.yml