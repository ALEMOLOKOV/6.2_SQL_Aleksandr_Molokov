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




