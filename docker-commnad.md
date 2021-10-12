# Docker 명령어

Docker의 명령어를 정리했다.

## Docker Hub에 로그인/로그아웃

### Docker 로그인

docker login 명령을 사용해 docker image repository에 로그인한다.

```shell
$ docker login [URL]
```

URL을 생략하면 Docker Hub에 로그인한다.

docker user id 변수 지정해놓기

```shell
$ export DOCKER_ID_USER="latte"
```

### Docker Hub에서 로그아웃

```shell
docker logout [서버명]
```

```shell
docker logout
```

## 이미지 관련

### 이미지 목록 보기

다운로드된 이미지들을 볼 수 있다.

```shell
$ sudo docker images [옵션] [repository명]
```

**자주 사용되는 옵션**

* \-a 모든 이미지
* \--digests : digest 항목도 함께 표시
* \-no-trunc : 모든 결과 표시
* \-q : Docker 이미지 id만을 표시

자신의 PC에 다운로드되거나 생성된 이미지들을 볼 수 있다.

```shell
root@instance-20210604-1640:~# docker images
REPOSITORY   TAG           IMAGE ID       CREATED             SIZE
ubuntu       git2          a319f7f07c4f   52 minutes ago      204MB
ubuntu       git           298e0e99d5e4   About an hour ago   204MB
ubuntu       git-layer-1   bbbcb883d109   About an hour ago   102MB
nginx        no_tar        d1f7b469f65f   2 hours ago         133MB
nginx        latest        4f380adfc10f   5 days ago          133MB
ubuntu       focal         9873176a8ff5   10 days ago         72.7MB
```

### 이미지 검색

```shell
$ sudo docker search [이미지 이름]
```

nginx 이미지를 검색한다. OFFICIAL이 'OK'인 것은 공식 이미지이다.

```shell
root@instance-20210604-1640:~#  sudo docker search nginx
NAME                               DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                              Official build of Nginx.                        15078     [OK]
jwilder/nginx-proxy                Automated Nginx reverse proxy for docker con…   2039                 [OK]
richarvey/nginx-php-fpm            Container running Nginx + PHP-FPM capable of…   814                  [OK]
... 생략 ...
```

### 이미지 받기

Docker Hub에서 이미지를 받으려면 pull을 사용한다.

```
$ docker image pull [옵션] <이미지명>[:태그명]
```

태그명을 생략하면 기본적으로 가장 최신 버전(latest)를 다운로드한다. 모든 태그의 이미지를 받으려면 '-a' 옵션을 준다. 이 경우 태그명을 지정할 수 없다.

latest 를 쓰면 최신 버전으로 받을수 있다.

```shell
sudo docker pull nginx:latest
```

docker pull 명령어에 이미지 URL을 지정할 수도 있는데, 이 때 http://는 생략한다. URL을 생략하면 로그인된 레파지토리에서 pull을 시도한다.

```shell
$ sudo docker pull nginx:latest
latest: Pulling from library/nginx
b4d181a07f80: Pull complete
edb81c9bc1f5: Pull complete
b21fed559b9f: Pull complete
03e6a2452751: Pull complete
b82f7f888feb: Pull complete
5430e98eba64: Pull complete
..
```

```shell
$ docker pull registry.hub.docker.com/ubuntu:5
```

### 이미지 삭제

```shell
$ sudo docker rmi [이미지 id]
```

docker images 명령으로 이미지들을 출력한다.

```shell
root@instance-20210604-1640:~# docker images
REPOSITORY   TAG           IMAGE ID       CREATED             SIZE
ubuntu       git2          a319f7f07c4f   About an hour ago   204MB
ubuntu       git           298e0e99d5e4   About an hour ago   204MB
ubuntu       git-layer-1   bbbcb883d109   About an hour ago   102MB
nginx        no_tar        d1f7b469f65f   3 hours ago         133MB
nginx        latest        4f380adfc10f   5 days ago          133MB
ubuntu       focal         9873176a8ff5   10 days ago         72.7MB
```

rmi 명령을 사용하여 삭제한다.

```shell
root@instance-20210604-1640:~# docker rmi d1f7b469f65f
Untagged: nginx:no_tar
Deleted: sha256:d1f7b469f65f0ec0d9ada3f2fac0c7e914c49554657f900788f47e606ee51e16
Deleted: sha256:919ca527c1961b443489a12041780cc99e0d09796dd64f20e3123234b920be46
root@instance-20210604-1640:~#
```

다시 image 명령으로 이미지가 삭제되었는지 확인한다.

```shell
root@instance-20210604-1640:~# docker images
REPOSITORY   TAG           IMAGE ID       CREATED             SIZE
ubuntu       git2          a319f7f07c4f   About an hour ago   204MB
ubuntu       git           298e0e99d5e4   About an hour ago   204MB
ubuntu       git-layer-1   bbbcb883d109   About an hour ago   102MB
nginx        latest        4f380adfc10f   5 days ago          133MB
ubuntu       focal         9873176a8ff5   10 days ago         72.7MB
```

no_tar가 삭제된 것을 확인할 수 있다.

#### 의존하는 자식 이미지 오류

이미지를 삭제할 때 다음과 같이 오류가 나고 이미지가 삭제 안되는 경우가 있다.

```shell
Error response from daemon: conflict: unable to delete 9873176a8ff5 (cannot be forced) - image has dependent child images
```

\-f 옵션을 사용하여 해결한다.

```
docker rmi 53d0b89b69b6 -f
```

#### 전체 이미지 삭제

아래의 경우 모든 이미지들을 삭제한다.

```shell
docker rmi $(docker images -a -q) -f
```

또는

```shell
$ docker image prune -a
```

#### dangling 이미지 삭제

Docker 를 사용하다 보면 위와 같이 : 이미지들이 쌓이는데 ( 이미지 생성과정에서 에러가 발생되면 쓸모없는 none 이미지가 남게된다. ) 이러한 이미지들을 한번에 정리할려고 하면 아래와 같이 명령어를 입력하면 된다.

```shell
root@instance-20210604-1640:~# docker image ls -a
REPOSITORY   TAG           IMAGE ID       CREATED       SIZE
ubuntu       git2          a319f7f07c4f   2 hours ago   204MB
<none>       <none>        410c5af75564   2 hours ago   102MB
ubuntu       git           298e0e99d5e4   2 hours ago   204MB
ubuntu       git-layer-1   bbbcb883d109   2 hours ago   102MB
nginx        latest        4f380adfc10f   5 days ago    133MB
ubuntu       focal         9873176a8ff5   10 days ago   72.7MB
```

```shell
docker rmi $(docker images -f "dangling=true" -q)
```

또는 prune 명령을 사용한다. prune 은 도커 측에서 공식적으로 제공하는 제거 명령어다. 이미지, 컨테이너, 볼륨, 네트워크 등등을 세분화해서 제거할 수 있다. 안전을 위해 기본적으로는 사용되지 않는 리소스만 제거하는 가비지 컬렉터 개념으로 보면 좋다. -a 플래그를 이용해 전체 리소스를 제거한다 해도 여러 리소스와 관련되어 있는 것들은 보호된다. 방금 위에서 소개한 row-level 명령어보다 비교적 안전하다.

```shell
docker image prune
```

### Docker Hub에 이미지 업로드

```shell
docker image push <이미지명>[:태그명]
```

### 이미지 태그 설정

```shell
docker image tag <기존 이미지명>[:태그명] <Docker Hub 사용자명>/<이미지명>:[태그명]
docker image tag alpine:latest honeyblock/alpine:1.0
```

### 이미지 검사

내려받은 이미지의 세부정보를 확인한다.

```shell
docker inspect [옵션] <컨테이너 이름 or 이미지 이름, ID>
```

실행 결과는 JSON형식으로 출력된다. --format 옵션으로 원하는 형태의 JSON형식으로 출력할 수도 있다.

```shell
root@instance-20210604-1640:~# docker inspect ubuntu:git2
[
    {
        "Id": "sha256:a319f7f07c4f1bba7e7d7ab0664b028b093c38324d11dfac893619975cd64cbe",
        "RepoTags": [
            "ubuntu:git2"
        ],
        "RepoDigests": [],
        "Parent": "sha256:410c5af75564c7b9c5405274d9db7e43b9ca4d341c3a80de4299e359e82e74e9",
        "Comment": "",
        "Created": "2021-06-28T07:07:59.501560913Z",
        "Container": "2b4bc526e1bc6ec848f9090752f72722af7a6d6be4daac1ff8f0bb6e6510294f",
        "ContainerConfig": {
            "Hostname": "",
...생략...
```

## 컨테이너 관련

### 컨테이너 목록 보기

#### 실행중인 컨테이너 목록

아래 명령을 실행하면 실행중인 컨테이너 목록을 볼 수 있다.

```shell
$ sudo docker ps
```

#### 모든 컨테이너 목록 보기

**옵션** -a : 모든 컨테이너 목록 출력

docker container ls -a 명령어를 입력하자. ls 명령어에 -a 옵션을 붙이면 실행 중인 컨테이너뿐만 아니라 종료된 컨테이너도 모두 표시한다.

```shell
root@instance-20210604-1640:~# docker ps -a
CONTAINER ID   IMAGE                COMMAND                  CREATED       STATUS                        PORTS     NAMES
aa807b0d1db1   ubuntu:git2          "bash -c 'git --vers…"   2 hours ago   Exited (0) 2 hours ago                  nervous_ptolemy
a8d59e1a03c6   ubuntu:git           "bash -c 'git --vers…"   2 hours ago   Exited (0) 2 hours ago                  adoring_curran
aa27217ac3da   ubuntu:git-layer-1   "/bin/sh -c 'apt-get…"   2 hours ago   Exited (0) 2 hours ago                  nice_hypatia
603e99977419   ubuntu:focal         "/bin/sh -c 'apt-get…"   2 hours ago   Exited (0) 2 hours ago                  clever_proskuriakova
14f80d300a86   nginx                "/docker-entrypoint.…"   4 hours ago   Exited (129) 39 minutes ago             hopeful_poincare
2849d40db447   nginx:latest         "/docker-entrypoint.…"   5 hours ago   Exited (129) 3 hours ago                strange_knuth
eddcd4663df9   nginx:latest         "/docker-entrypoint.…"   6 hours ago   Exited (0) 6 hours ago                  quirky_pare
```

**STATUS**\
**UP** 실행중 **Exited** Docker Process에 나타나는 Exit Code 들은 Bash Script의 Exit Code를 따라가며, 0은 성공, 1 \~ 255는 실패를 의미한다. 어떠한 이유에서든 Exited Code가 발생한 Container는 더 이상 사용이 불필요한 상태이다. Exited Status 상태의 Container들을 한번에 정리하기 위해 아래와 같은 명령어를 실행한다.

> exit(0)은 정상적인 컨테이너이므로 실제 운영중일 때에는 삭제하지 말아야 한다.

```shell
docker rm -v $(docker ps -a -q -f status=exited)
```

#### 필터

\-f 옵션을 사용하면 조회 결과를 특정 조건에 따라 필터링해서 볼 수 있다. 예를 들어, 특정 이미지로 부터 만들어진 컨테이어만 보고 싶다면 ancestor 필터를 사용하면 된다.

```shell
docker ps -af "ancestor=python:alpine"
```

#### 디스크 용량

\-s 옵션을 사용하면 각 컨테이너의 디스크 사용량까지 볼 수 있다.

```shell
docker ps -s
```

### 컨테이너 실행

도커를 사용한다는 것은 결국 이미지를 이용해서 컨테이너를 생성하고 실행한다는 것이다. docker run 명령어를 사용하면 컨테이너를 실행할 수 있다.

```shell
$ sudo docker run [options] image[:TAG|@DIGEST] [COMMAND] [ARG...]
```

실행 옵션

* \-d detached mode. 백그라운드 모드
* \-p 호스트와 컨테이너의 포트를 연결 (포워딩)
* \-v 호스트와 컨테이너의 디렉토리를 연결 (마운트)
* \-e 컨테이너 내에서 사용할 환경변수 설정
* \--name 컨테이너 이름 설정
* \--it -i와 -t를 동시에 사용한 것. 터미널 입력을 위한 옵션 (컨테이너의 표준 \* 입력과 로컬 컴퓨터의 키보드 입력을 연결)
* \--rm 프로세스 종료시 컨테이너 자동 제거
* \--link 컨테이너 연결 \[컨테이너 명:별칭]

#### -d 옵션

백그라운드로 실행하려면 **-d** 옵션을 사용한다. -d 옵션을 사용하면 컨테이너가 detached 모드에서 실행되며, 실행 결과로 컨테이너 ID만을 출력한다.

```shell
# docker container run --name webserver -d -p 8080:80 nginx:latest
9f05fb54e86b4f3542a028708b0193d33be32fb4831b4393f5fcf35327a0437c
```

ps 명령어로 확인해보면 백그라운드로 실행중인 것을 볼 수 있다.

```shell
root@instance-20210604-1640:~# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                   NAMES
9f05fb54e86b   nginx:latest   "/docker-entrypoint.…"   34 seconds ago   Up 32 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   webserver
```

명령어를 -d 옵션없이 실행했다면, 해당 터미널에서 Ctrl + C를 눌러서 빠져나오는 순간 해당 컨테이너는 종료될 것이다.

> 컨테이너를 run 명령을 사용해서 실행했다면 컨테이너가 등록된다. 다음부터는 start 명령으로 실행해야 한다.

#### -it 옵션

\-i 옵션과 -t 옵션은 같이 쓰이는 경우인데, 이 두 옵션은 컨테이너를 종료하지 않은체로, 터미널의 입력을 계속해서 컨테이너로 전달하기 위해서 사용한다. -it 옵션은 특히 컨테이너의 쉘(shell)이나 CLI 도구를 사용할 때 매우 유용하게 사용된다.

```shell
docker run -it  nginx:latest /bin/sh
```

다음은 bash 셸을 실행한다. 프롬프트가 표시되면 ls 명령을 실행했다.

```shell
# docker run -it nginx:latest /bin/sh
# ls
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

빠져나갈 때에는 exit를 입력한다.

#### --name 옵션

\--name 옵션을 사용하면 컨테이너 이름을 직접 지정할 수 있다. 다음은 --name 옵션의 사용 예를 보여준다. docker kill 커맨드로 해당 컨테이너를 종료하거나, docker rm 커맨드로 해당 컨테이너를 삭제할 때 컨테이너 이름을 컨테이너 ID 대신에 사용할 수 있다.

```shell
docker run -d --name webserver -p 8080:80 nginx:latest
```

#### -e 옵션

Docker 컨테이너의 환경변수를 설정하기 위해서는 -e 옵션을 사용한다. 또한, -e 옵션을 사용하면 Dockerfile의 ENV 설정도 덮어쓴다. 아래 커맨드는 FOO 환경 변수를 bar로 세팅을 하고, 환경 변수를 출력한다.

```shell
$ docker run -e FOO=bar nginx:latest env
```

```shell
# docker run -e FOO=bar nginx:latest env
HOSTNAME=b10ca907e33d
HOME=/root
PKG_RELEASE=1~buster
NGINX_VERSION=1.21.0
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
FOO=bar
NJS_VERSION=0.5.3
PWD=/
```

#### -p 옵션

\-p 옵션은 호스트와 컨테이너 간의 포트(port) 배포(publish)/바인드(bind)를 위해서 사용된다..호스트(host) 컴퓨터에서 컨테이너에서 리스닝하고 있는 포트로 접속할 수 있도록 설정해준다. 아래 커맨드는 컨테이너 내부에서 8080 포트로 리스닝하고 있는 HTTP 서버를 호스트 컴퓨터에서 80 포트로 접속할 수 있도록 해준다.

```shell
docker run -d --name webserver -p 8080:80 nginx:latest
```

#### —rm 옵션

\--rm 옵션은 컨테이너를 일회성으로 실행할 때 주로 쓰인다. 컨테이너가 종료될 때 컨테이너와 관련된 리소스(파일 시스템, 볼륨)까지 깨끗이 제거해준다.

```shell
docker run --rm  -p 8080:80 nginx:latest /bin/sh
```

#### 명령 전달

docker run 명령에서 마지막에 주는 인자 값은 이 컨테이너가 실행할 명령을 전달하는 인자이다. 우분투 컨테이너의 경우 아무런 값을 입력하지 않으면 "/bin/bash"가 실행되고, 인자를 전달하면 그 값이 실행된다.

아래 명령은 'ls'를 실행한다.

```shell
docker run ubuntu:focal /bin/sh -c 'ls' 
```

아래 명령은 'apt-get update'를 실행한다.

```shell
docker run ubuntu:focal /bin/sh -c 'apt-get update' 
```

아래 명령은 git를 설치한다.

```
docker run ubuntu:git-layer-1 /bin/sh -c 'apt-get install -y git'
```

### 컨테이너 시작

중지된 Docker 컨테이너를 다시 시작하려면 docker start 커맨드를 사용한다. 마찬가지로 재시작하고 싶은 컨테이너의 아이디나 이름을 인자로 넘기면 된다.

> run 명령으로 컨테이너를 실행하면 새로운 컨테이너가 등록된다. 등록된 컨테이너를 실행하려면 start 명령을 사용한다. docker 컨테이너를 실행한 다음 동일한 명령을 실행하면 다음과 같은 에러가 나타나는데 이것은 docker run 명령이 "create" 와 "start" 명령을 한번에 실행시키는 명령이기 때문에 create 시 이미 동일한 이름의 컨테이너가 존재하기 때문에 발생하는 문제이다. 이 경우 docker rm 으로 삭제한 후 다시 실행하면 된다.

```shell
$ sudo docker start [컨테이너 id 또는 name]
```

### 컨테이너 재시작

```shell
$ sudo docker restart [컨테이너 id 또는 name]
```

### 컨테이너 접속

```shell
$ sudo docker attach [컨테이너 id 또는 name]
```

\-it 옵션을 사용하여 무중단으로 컨테이너에서 빠져나올 수 있고, /bin/bash를 통해 컨테이너 내에 있는 bash를 실행하여 컨테이너에 접속할 수 있다.

```shell
docker exec -it [container name] /bin/bash
```

**exec와 run 명령의 차이점**\
exec는 실행준인 컨테이너에 명령을 전달하고 run은 새로운 컨테이너를 만들어서 실행한다는 차이점이 있다.

### 컨테이너 정지

```shell
$ sudo docker stop [컨테이너 id 또는 name]
```

* Ctrl + D를 입력하면 컨테이너가 정지된다.
* Ctrl + P, Ctrl + Q를 차례대로 입력하여 컨테이너를 정지하지 않고, 컨테이너에서 빠져나온다.

### 컨테이너 삭제

```shell
$ sudo docker rm [컨테이너 id 또는 name]
```

#### 모든 컨테이너 삭제

```shell
$ sudo docker rm `docker ps -a -q`
```

#### 특정 컨테이너 제거

```shell
$ docker rm <컨테이너 아이디 or 이름>
```

#### 정지된 컨테이너 제거

```shell
$ docker container prune
```

## docker cp 명령어

docker는 HOST와 PC간의 파일 이동을 위해 복사 명령어인 cp를 지원한다. docker cp 명령어는 호스트에서 컨테이너로, 컨테이너에서 호스트로 양 방향 모두를 지원하는 명령어이다.파일이 아닌 디렉터리 경로를 지정한 경우 디렉터리 전체를 통채로 복사한다.

```shell
docker cp [host 파일경로] [container name]:[container 내부 경로]
```

```shell
docker cp /mnt/e/tmp/examples was:/usr/local/tomcat/webapps
```

## sudo 없이 Docker 명령어 쓰기

```shell
$ sudo usermod -aG docker $USER # 현재 접속중인 사용자에게 권한주기
```

```shell
$ sudo usermod -aG docker a-user # a-user 사용자에게 권한주기
```

사용자가 로그인 중 일 때, 다시 로그인하면 권한이 적용된다.
