# 6.2_SQL_Aleksandr_Molokov

Задание 1

Создание каталога для хранения данных
vagrant@vagrant:~$ mkdir -p $HOME/docker/volumes/postgres

Загрузка postgres
vagrant@vagrant:~$ sudo docker pull postgres:12

Docker-compose.yaml

version: '3'
services:
 db:
   container_name: pg12
   image: postgres:12
   environment:
     POSTGRES_USER: amolokov
     POSTGRES_PASSWORD: november2022
     POSTGRES_DB: postgres_db
   ports:
     - "5432:5432"
   volumes:
     - db_vol1:/home/database/
     - backup_vol2:/home/backup/

volumes:
 db_vol1:
 backup_vol2:

Развертывание docker-compose  и запуск в нутри него bash

vagrant@vagrant:~/docker/volumes/postgres$ sudo docker-compose up -d

vagrant@vagrant:~/docker/volumes/postgres$ sudo docker exec -it pg12 bash
root@a43834a670f0:/#

Задание 2

Создание Базы данных test_db
root@a43834a670f0:/# createdb test_db -U amolokov

Подключение к базе
root@a43834a670f0:/# psql -d test_db -U amolokov
psql (12.13 (Debian 12.13-1.pgdg110+1))
Type "help" for help.
test_db=#

Создание пользователя test-admin_user
test_db=# CREATE USER test_admin_user;
CREATE ROLE
test_db=#

Создание таблиц orders и clients в БД test_db

test_db=# CREATE TABLE orders
(
   id SERIAL PRIMARY KEY,
   наименование TEXT,
   цена INTEGER
);
CREATE TABLE
test_db=#

test_db=# CREATE TABLE clients
(
    id SERIAL PRIMARY KEY,
    фамилия TEXT,
    "страна проживания" TEXT,
    заказ INTEGER,
    FOREIGN KEY (заказ) REFERENCES orders(id)
);
CREATE TABLE
test_db=#

test_db=# CREATE INDEX country_index ON clients ("страна проживания");
CREATE INDEX
test_db=#

Предоставление привилегий на все операции пользователю test-admin-user на таблицы БД test_db

test_db=# GRANT ALL ON TABLE orders TO test_admin_user;
GRANT
test_db=# GRANT ALL ON TABLE clients TO test_admin_user;
GRANT
test_db=#

Создание пользователя test-simple-user

test_db=# CREATE USER test_simple_user;
CREATE ROLE
test_db=#

Предоставление пользователю test-simple-user правв на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

test_db=# GRANT SELECT,INSERT,UPDATE,DELETE ON TABLE orders TO test_simple_user;
GRANT
test_db=# GRANT SELECT,INSERT,UPDATE,DELETE ON TABLE clients TO test_simple_user;
GRANT
test_db=#

Итоговый список БД
![Задание 2 итоговый список БД](https://user-images.githubusercontent.com/109212419/204089546-354fae66-eed4-466f-a23f-508037116482.jpg)

Описание таблиц
![Задание 2 таблица orders](https://user-images.githubusercontent.com/109212419/204089586-e60825c9-0e1c-4a65-a2b1-3c45c052bab7.jpg)
![Задание 2 таблица clients](https://user-images.githubusercontent.com/109212419/204089592-2d6ac019-c7ec-4ee6-b59f-9ad3ecf4f125.jpg)


Список пользователей с правами над таблицами

test_db=# SELECT grantee, table_catalog, table_name, privilege_type FROM information_schema.table_privileges WHERE table_name IN ('orders','clients');
![Задание 2 Список пользователей с правами над таблицами](https://user-images.githubusercontent.com/109212419/204089514-9e2ff2e9-a6b9-4b72-a442-e865236a587a.jpg)










