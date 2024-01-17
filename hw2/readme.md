Описание/Пошаговая инструкция выполнения домашнего задания:

1. создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом
поставить на нем Docker Engine
```bash
# поставим докер -- https://docs.docker.com/engine/install/ubuntu/ 
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER && newgrp docker
# Создаем docker-сеть: 
sudo docker network create pg-net
```
2. сделать каталог /var/lib/postgres
```shell
sudo mkdir /var/lib/postgres
```
3. развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql 
```bash
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15 
```
4. развернуть контейнер с клиентом postgres
```shell
focus@postgres-hw-2:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED              STATUS              PORTS                                       NAMES
dddaa4f2edec   postgres:15   "docker-entrypoint.s…"   About a minute ago   Up About a minute   5432/tcp                                    pg-client
5ec8356a01b0   postgres:15   "docker-entrypoint.s…"   5 minutes ago        Up 5 minutes        0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-server
 
```
5. подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
```shell
# Запускаем отдельный контейнер с клиентом в общей сети с БД: 
sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres 

postgres=# create table persons (user_id int);
CREATE TABLE
postgres=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner   
--------+---------+-------+----------
 public | persons | table | postgres
(1 row)

postgres=# insert into persons values (1), (2) , (3);
INSERT 0 3
postgres=# select * from persons ;
 user_id 
---------
       1
       2
       3
(3 rows)

```
6. подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера
```shell
➜  ~ psql -h ${YA_HOST} -p 5432 -U postgres 
Password for user postgres: 
psql (14.10 (Ubuntu 14.10-0ubuntu0.22.04.1), server 15.5 (Debian 15.5-1.pgdg120+1))
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
Type "help" for help.

postgres=# 

```
7. удалить контейнер с сервером
```shell
focus@postgres-hw-2:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
dddaa4f2edec   postgres:15   "docker-entrypoint.s…"   13 minutes ago   Up 13 minutes   5432/tcp                                    pg-client
5ec8356a01b0   postgres:15   "docker-entrypoint.s…"   17 minutes ago   Up 17 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-server
focus@postgres-hw-2:~$ sudo docker stop 5ec8356a01b0
5ec8356a01b0
focus@postgres-hw-2:~$ sudo docker rm 5ec8356a01b0
5ec8356a01b0
focus@postgres-hw-2:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS      NAMES
dddaa4f2edec   postgres:15   "docker-entrypoint.s…"   13 minutes ago   Up 13 minutes   5432/tcp   pg-client

```
8. создать его заново
```bash
focus@postgres-hw-2:~$ sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
a3e0108433218a9532373bd69d32d0719cd4997bc8b65b5abfb2d8b2016f4240
focus@postgres-hw-2:~$ sudo docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
a3e010843321   postgres:15   "docker-entrypoint.s…"   3 seconds ago    Up 2 seconds    0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-server
dddaa4f2edec   postgres:15   "docker-entrypoint.s…"   14 minutes ago   Up 14 minutes   5432/tcp                                    pg-client

```
9. подключится снова из контейнера с клиентом к контейнеру с сервером
10. проверить, что данные остались на месте
```shell
focus@postgres-hw-2:~$ sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres 
Password for user postgres: 
psql (15.5 (Debian 15.5-1.pgdg120+1))
Type "help" for help.

postgres=# select * from persons ;
 user_id 
---------
       1
       2
       3
(3 rows) 
```
оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами