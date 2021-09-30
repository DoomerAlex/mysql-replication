### Запуск приложения
1. Собрать контейнеры:
```shell
docker-compose build
```
2. Запустить контейнеры:
```shell
docker-compose up -d
```
### Настройка слейва:
1. Зайти в master базу:
```shell
docker-compose exec mysql-master mysql -u root -p

pass: 74657465
```
2. Создать пользователя под репликацию
```mysql
CREATE USER 'replication'@'%';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
```
3. Убеждаемся, что двоичный журнал включен
```mysql
SHOW MASTER STATUS;

+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      156 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
```
4. Заходим на slave базу:
```shell
docker-compose exec mysql-slave mysql -u root -p

pass: 74657465
```
5. Указываем slave-у откуда будем делать репликацию данных.
   MASTER_LOG_FILE - берем из шага 3 в столбце File
   MASTER_LOG_POS - берем из столбца 156
```mysql
 CHANGE MASTER TO MASTER_HOST='mysql-master', MASTER_USER='replication', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=156;
```
6. Запускаем репликацию:
```mysql
START SLAVE;
```
### Проверка состояния slave-а:
1. Зайти в MySQL в slave контейнере:
```shell
docker-compose exec mysql-slave mysql -u root -p

pass: 74657465
```
2. Проверить статус слейва:
```mysql
SHOW SLAVE STATUS\G
```