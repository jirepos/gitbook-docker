# Docker에서 NginX 설치

Docker로 Nginx를 설치하고 설정파일을 변경하는 것을 알아본다.

## NginX 설치

### 이미지 다운로드

```shell
sudo docker pull nginx
```

이미지를 다운로드 받는다.

```shell
ubuntu@DESKTOP-POHP5V7:~$ sudo docker pull nginx
[sudo] password for ubuntu:
Using default tag: latest
latest: Pulling from library/nginx
b4d181a07f80: Pull complete
edb81c9bc1f5: Pull complete
b21fed559b9f: Pull complete
03e6a2452751: Pull complete
b82f7f888feb: Pull complete
5430e98eba64: Pull complete
Digest: sha256:47ae43cdfc7064d28800bc42e79a429540c7c80168e8c8952778c0d5af1c09db
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

## 이미지 확인

```shell
sudo docker images
```

다운로드된 이미지들을 볼 수 있다.

```shell
ubuntu@DESKTOP-POHP5V7:~$ sudo docker images
REPOSITORY          TAG       IMAGE ID       CREATED        SIZE
docker101tutorial   latest    2840e5e933eb   21 hours ago   28MB
nginx               latest    4f380adfc10f   47 hours ago   133MB
alpine/git          latest    b8f176fa3f0d   4 weeks ago    25.1MB
ubuntu@DESKTOP-POHP5V7:~$
```

## NginX 실행

```shell
docker container run --name webserver -d -p 8080:80 nginx
```

\--name 실행할 컨테이너의 이름을 지정한다.

\-d 백그라운드로 실행한다.

\-p 포트를 설정한다. 왼쪽에 호트트의 포트를 쓰고 오른쪽에 컨테이너의 포트를 설정한다.

```shell
-p 8080:80
```

docker가 실행되는 호스트에서 80 번을 다른 프로세스가 사용한다고 가정하고 브라우저에서 호스트로 연결할 때는 8080 포트로 연결하도록 설정한다. 컨테이너의 NginX는 80번을 사용하기 때문에 포트를 연결하기 위해서 8080:80과 같이 설정한다.

정상완료되면 컨테이너 ID가 나타난다.

```shell
ubuntu@DESKTOP-POHP5V7:~$ docker container run --name webserver -d -p 8080:80 nginx
b96b4d0dc36a892ef972e92f056ed5d8a5323f1feb4f25edb0d727200a893a1a
```

### 컨테이너 확인

실행 중인 컨테이너를 확인한다.

```shell
sudo docker container ps
```

프로세스를 확인할 수 있다.

```shell
ubuntu@DESKTOP-POHP5V7:~$ sudo docker container ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                                   NAMES
b96b4d0dc36a   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp, :::8080->80/tcp   webserver
```

### Nginx 실행 확인

브라우저에서 http://localhost:8080으로 접속한다. 다음과 같이 Welcome 화면이 나올 것이다.

![](<.gitbook/assets/image (13).png>)

### Nginx 정지

```shell
docker stop webserver
```

정지된 컨테이너는 ps -a로 조회해야 한다.

```shell
sudo docker ps -a
```

```shell
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS                    PORTS                                   NAMES
b96b4d0dc36a   nginx        "/docker-entrypoint.…"   6 minutes ago   Up 6 minutes              0.0.0.0:8080->80/tcp, :::8080->80/tcp   webserver
e5916ffc0744   alpine/git   "git clone https://g…"   21 hours ago    Exited (0) 21 hours ago                                           repo
```

### Docker 컨테이너 실행

```shell
docker start webserver
```

## index.html 생성

\-v 옵션을 주어 경로를 마운트 시킨다.

```shell
docker container run -v /home/ubuntu/html --name webserver -d -p 8080:80 nginx
```

## nginx.conf 파일 변경 설정하기

사용자 홈 디렉토리에 nginx 디렉토리 만들고 그 아래 conf 디렉토리 만든다. 그 아래에 nginx.conf 파일을 생성한다.

```shell
📁nginx
  📁conf
     📄 nginx.conf
```

다음과 같이 작성한다.

```shell
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

    #security
    server_tokens off;
}
```

nginx 디렉토리에 Dockerfile을 만든다.

```shell
📁nginx
  📄 Dockerfile
  📁conf
     📄 nginx.conf
```

다음과 같이 작성한다.

```shell
#Dockerfile
FROM nginx:latest

COPY config/nginx.conf /etc/nginx/conf.d/nginx.conf

CMD ["nginx", "-g", "daemon off;"]

EXPOSE 80
```

Docker 이미지를 생성한다.

```shell
docker build --tag nginx-test:1.0 .
```

생성한 Docker 이미지를 확인한다.

```shell
docker images
```

이미지를 실행한다.

```shell
docker run nginx-test:1.0
```
