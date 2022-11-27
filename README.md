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


Задание 3

Наполнение таблиц данными

test_db=# INSERT INTO orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);

INSERT 0 5

test_db=# INSERT INTO clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), (4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');

INSERT 0 5


test_db=#

SQL запросы для вычисления количества записей в таблицах

![Задание 3 результат кол-во записей в таблицах ](https://user-images.githubusercontent.com/109212419/204155271-e855f4c7-aae8-4482-b5c6-651efd67c622.jpg)

ЗАДАНИЕ 4

Связь записей в таблицах

test_db=#  UPDATE clients SET заказ=(select id from orders where наименование='Книга') WHERE фамилия='Иванов Иван Иванович';

UPDATE 1

test_db=# UPDATE clients SET заказ=(select id from orders where наименование='Монитор') WHERE фамилия='Петров Петр Петрович';

UPDATE 1

test_db=# UPDATE clients SET заказ=(select id from orders where наименование='Гитара') WHERE фамилия='Иоганн Себастьян Бах';

UPDATE 1

Запрос для выдачи всех пользователей которые совершили заказ

test_db=# SELECT* FROM clients WHERE заказ IS NOT NULL;

![Задание 4 все кто сделал заказ](https://user-images.githubusercontent.com/109212419/204155721-ae97b7ae-780f-4cba-abc0-141fdf73ebe9.jpg)

ЗАДАНИЕ 5

![Задание 5 explain](https://user-images.githubusercontent.com/109212419/204155835-62cb6986-f481-4246-9a04-9505efe1ee49.jpg)

Чтение данных из таблицы clients происходит с использованием метода Seq Scan — последовательного чтения данных. Значение 0.00 — ожидаемые затраты на получение первой строки. Второе — 18.10 — ожидаемые затраты на получение всех строк. rows - ожидаемое число строк, которое должен вывести этот узел плана. При этом так же предполагается, что узел выполняется до конца. width - ожидаемый средний размер строк, выводимых этим узлом плана (в байтах). Каждая запись сравнивается с условием "заказ" IS NOT NULL. Если условие выполняется, запись вводится в результат. Иначе — отбрасывается.

ЗАДАНИЕ 6

Создание бэкапа БД test_db и сохранение его в volume, предназначенный для бэкапов

root@a43834a670f0:/# pg_dump -U amolokov test_db > /home/backup/test_db.backup |echo $?
0

root@a43834a670f0:/#

Остановка контейнера с PostgreSQL (без удаления volumes).

![Задание 6 остановка контейнера pg12](https://user-images.githubusercontent.com/109212419/204156329-c32eab3b-8ff1-4415-9811-7eefbc593f48.jpg)


Запуск нового пустого контейнера с PostgreSQL.

vagrant@vagrant:~/docker/volumes/postgres$ sudo docker run --name new_pg12 -e POSTGRES_PASSWORD=123456 -d postgres:12

![Задание 6 создание и запуск нового контейнера](https://user-images.githubusercontent.com/109212419/204156520-4b348a7a-93ab-4398-bc4c-9fde73b0644f.jpg)


Восстановление БД test_db в новом контейнере new_pg12.

Копирование файла из контейнера pg12 в new_pg12

![Задание 6 оибка при копировании данных](https://user-images.githubusercontent.com/109212419/204158844-38b19e84-16d9-407b-9630-d2cd982dce28.jpg)

Пока разбираюсь















