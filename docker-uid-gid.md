# Docker UID/GID 문제

mairadb container에 접속해서 mysql 계정의 UID/GID를 확인해 보면 999:999이다.

```shell
#> docker exec -it mariadb bash
root@413d86d9240d:/# cat /etc/passwd
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
mysql:x:999:999::/home/mysql:/bin/sh
```

host 시스템의 999계정을 확인해 보면 systemd-coredump 계정이 999이다.

```shell
#> cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
opc:x:1000:1000::/home/opc:/bin/sh
ubuntu:x:1001:1001:Ubuntu:/home/ubuntu:/bin/bash
```

docker로 mariadb 컨테이너를 생성하면 지정한 볼륨 디렉터리에 소유자가 systemd-coredump가 된다.

```shell
#> ls -al
total 32
drwxr-xr-x 4 root             root 4096 Aug  3 00:40 .
drwxr-xr-x 5 root             root 4096 Aug  2 10:13 ..
drwxr-xr-x 2 root             root 4096 Aug  3 00:25 conf.d
-rw-r--r-- 1 root             root  137 Jul 21 00:13 create_db.sql
drwxr-xr-x 5 systemd-coredump root 4096 Aug  3 00:41 data
```

Dockerfile을 빌드할 때 usermod와 groupmod를 통해 postgres의 uid/gid를 일반 사용자의 uid/gid로 매핑을 한다.

```shell
groupmod -g 600 group01
usermod -u 900 -g 600 user01
```

## References

[Understanding how uid and gid work in Docker containers](https://medium.com/@mccode/understanding-how-uid-and-gid-work-in-docker-containers-c37a01d01cf)

[Docker-compose.yml 설정](https://dev.to/acro5piano/specifying-user-and-group-in-docker-i2e)
