# Docker에서 Mariadb 설치

## 도커 이미지 다운로드

pull 명령을 사용하여 mariadb를 내려받는다.

```shell
docker pull mariadb
```

```shell
#> docker pull mariadb
Using default tag: latest
latest: Pulling from library/mariadb
Digest: sha256:0c72b63198ac53df4e84db821876c73794b00509b2d8a77100d186a13e49ac31
Status: Image is up to date for mariadb:latest
docker.io/library/mariadb:latest
```

## 컨테이너 실행

### MariaDB 설정

d:/docker/maridb/conf.d 디렉터리를 만든다. 거기에 my.cnf 파일을 생성하고 다음과 같이 입력한다. Mariadb가 시작할 때 /etc/mysql/my.cnf 파일의 설정을 사용하는데 /etc/mysql/conf.d에 설정이 있으면 이 두가지를 합친다. 그러나 conf.d에 있는 설정과 /etc/my.cnf 설정이 중복되면 conf.d에 있는 것이 우선한다. 그래서 컨테이너 시작시에 호스트 시스템에 my.cnf 파일을 읽도롤 마운트 시켜야 한다.

> 시작 설정은 /etc/mysql/my.cnf 파일에 정의된다. 그리고 파일은 /etc/mysl/conf.d 디렉터리에서 발견된 .cnf로 끝나는 파일들을 포함한다. 이 디렉터리에서 Settings은 /etc/mysql/my.cnf에 설정들을 설정들을 추가하거나 덮어쓴다. MariaDB의 구성을 커스텀하려면 호스트 머신의 디렉터리에 대체할 구성 파일을 생성할 수 있다. mariadb container 내부에 /etc/mysql/conf.d로 그 디렉터리를 마운트할 수 있다.

```shell
[mysqld]
character-set-client-handshake = FALSE
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

[client]
default-character-set = utf8mb4

[mysql]

default-character-set = utf8mb4

[mysqldump]
default-character-set = utf8mb4
```

### 환경변수 설정

MariaDB의 root의 계정의 패스워드를 설정해야 한다. MYSQL_ROOT_PASSWORD를 사용한다. 컨테이너 시작 시에 환경변수를 설정할 수 있는데 MARIADB_ROOT_PASSWORD 또는 MYSQL_ROOT_PASSWORD를 설정할 수 있다. 사용자를 추가하려면 MARIADB_USER, MARIADB_PASSWORD 환경 변수들을 사용할 수 있다.

```shell
-e MYSQL_ROOT_PASSWORD=1234
```

MariaDB의 커스텀 설정 디렉터리인 /etc/mysql/conf.d를 호스트 시스템의 디렉터리를 사용하도록 -v 옵션을 사용한다.

```shell
-v /mnt/d/docker/mariadb/conf.d:/etc/mysql/conf.d
```

### Datbase 파일위치

MariaDB의 DB파일의 위치를 호스트 시스템의 파일시스템으로 설정하기 위해서 -v 옵션을 사용한다.

```shell
-v /mnt/d/docker/mariadb/data:/var/lib/mysql
```

### 컨테이너 실행 스크립트

이 모든 것을 합치면 다음과 같다. 이제 컨테이너를 정식으로 실행할 것이다.

```shell
#!/bin/bash
docker container run -d -p 3306:3306  \
 -e MYSQL_ROOT_PASSWORD=1234 \
 -v /mnt/d/docker/mariadb/conf.d:/etc/mysql/conf.d  \
 -v /mnt/d/docker/mariadb/data:/var/lib/mysql \
 --name mariadb mariadb:latest
```

## 로그 보기

```shell
$ docker logs -f --tail=10 mariadb
```

## 컨테이너 접속하기

```shell
docker exec -it mariadb /bin/bash
```

## mysql client 설치

mysql -V를 실행하면 설치할 수 있는 패키지들이 보여진다.

```shell
#> mysql -V

Command 'mysql' not found, but can be installed with:

apt install mysql-client-core-8.0     # version 8.0.25-0ubuntu0.20.04.1, or
apt install mariadb-client-core-10.3  # version 1:10.3.29-0ubuntu0.20.04.1c
```

mariadb-client를 설치한다.

```shell
apt install mariadb-client-core-10.3
```

mysql -V를 실행하여 확인한다.

```shell
#> mysql -V
mysql  Ver 15.1 Distrib 10.3.29-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2
```

## MariaDB 접속

Host 시스템에서 docker에서 실행중인 MariaDB에 접속하려면 다음과 같이 -h 옵션에 IP를 입력해야 한다.

```shell
#> mysql -u root -p -h 127.0.0.1 mysql
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 10.5.11-MariaDB-1:10.5.11+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [mysql]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.009 sec)

MariaDB [mysql]>
```

## DB 생성 및 사용자 생성

lattedb를 생성하고 latte 사용자를 생성한다.

### Database 생성

```sql
CREATE DATABASE lattedb CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
```

```sql
CREATE USER 'latte'@'localhost' IDENTIFIED BY '1234' PASSWORD EXPIRE NEVER; 
GRANT ALL PRIVILEGES ON lattedb.* TO 'latte'@'localhost';
flush privileges;
```

```sql
CREATE USER 'latte'@'%' IDENTIFIED BY '1234' PASSWORD EXPIRE NEVER; 
GRANT ALL PRIVILEGES ON lattedb.* TO 'latte'@'%';
flush privileges;
```

위 SQL을 하나의 파일로 만들고 다음과 같이 DB를 생성할 수 있다.

```sql
CREATE DATABASE lattedb CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
CREATE USER 'latte'@'localhost' IDENTIFIED BY '1234' PASSWORD EXPIRE NEVER; 
GRANT ALL PRIVILEGES ON lattedb.* TO 'latte'@'localhost';
flush privileges;
CREATE USER 'latte'@'%' IDENTIFIED BY '1234' PASSWORD EXPIRE NEVER; 
GRANT ALL PRIVILEGES ON lattedb.* TO 'latte'@'%';
flush privileges;
```

다음과 같이 사용한다.

```shell
mysql -u root -p -h 127.0.0.1 < create.sql
```

## 권한

현재사용자의 권한 보기

```sql
show grants for current_user; 
```

특정 사용자의 권한 보기

```sql
show grants for 'root'@'%';
```
