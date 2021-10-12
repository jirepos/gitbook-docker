# Dockerfile

## 빠르게 시작하기

### Dockerfile 작성

FROM은 새로운 이미지를 생성할 때 기반으로 사용할 이미지를 지정한다. 아래 코드는 ubuntu:latest 이미지를 기반 이미지로 사용한다.

```shell
FROM ubuntu:latest
ENTRYPOINT ["echo", "hello"]
```

ENTRYPOINT는 컨테이너를 시작할 때 실행할 명령어를 입력한다. 위 코드는 'echo hello'를 실행 명령어로 사용한다.

### 빌드하기

Dockerfile을 작성했으면 docker build 명령어로 이미지를 생성할 수 있다.

```shell
docker build --tag ubuntu:my-test .
```

\-tag(또는 -t) 옵션은 새로 생성할 이미지 이름을 지정한다. 리파티터리 이름으로 ubuntu를 사용하고 태그로 my-test를 사용했다. 마지막의 점(.)은 Dockerfile의 위치를 지정한 것인데 .은 현재 경로이다. 파일 이름이 Dockerfile이 아닌 경우 --file 또는 -ㄹ 옵션을 사용해서 파일이름을 지정한다.

## Dockerfile로 이미지 만들기

Dockerfile과 같은 디렉터리에 있는 모든 파일들을 컨텍스트(context)라고 한다. docker build 를 사용하여 이미지를 생성할 때 컨텍스트 모두를 Docker 데몬에 전송하기 때문에 루트 디렉토리에 Dockerfile을 생성하면 시스템 처리 속도가 굉장히 느려질 수 있다. 때문에 Dockerfile은 따로 디렉토리를 새로 생성하는것으로 권장된다.

컨텍스트에서 파일이나 디렉토리를 제외할때 Dockerfile과 같은 경로에 .dockerignore 파일을 생성하여 제외하고싶은 파일이나 디렉토리를 지정해줄 수 있다.

```shell
Hello, test.txt
*.log
.git
```

### 새로운 이미지 만들기(FROM)

ubuntu 최신 이미지로부터 이미지를 만들려면 다음과 같이 작성한다.

```shell
FROM ubuntu:latest
```

**FROM**은 이미지를 생성할 때 사용할 기반 이미지를 지정한다. ubuntu:latest 이미지로 부터 이미지를 생성한다.

### 패키지 설치(RUN)

기반 이미지에 nginx 최신 이미지를 설치하려면 다음과 같이 작성한다. ubuntu 최신 이미지를 가져와서 그 이미지에 nginx를 설치한다. nginx 설치할 때 설치할 것인지 물어 보는데 -y 옵션을 준다.

```shell
FROM ubuntu:latest

RUN apt-get update
RUN apt-get install nginx -y
```

정상적으로 ubuntu이미지에 nginx를 설치하여 이미지를 생성하는 것을 확인할 수 있다.

```shell
root@instance-20210604-1640:~# docker build --tag ubuntu:nginx .
Sending build context to Docker daemon  26.11kB
Step 1/3 : FROM ubuntu:latest
latest: Pulling from library/ubuntu
c549ccf8d472: Pull complete
Digest: sha256:aba80b77e27148d99c034a987e7da3a287ed455390352663418c0f2ed40417fe
Status: Downloaded newer image for ubuntu:latest
 ---> 9873176a8ff5
Step 2/3 : RUN apt-get update
 ---> Running in 94b480c988e2
Fetched 18.4 MB in 7s (2511 kB/s)
Reading package lists...
Removing intermediate container 94b480c988e2
 ---> 6172afb552ca
Step 3/3 : RUN apt-get install nginx -y
 ---> Running in 5e8f967aaffc
Removing intermediate container 5e8f967aaffc
 ---> 59026582cf0d
Successfully built 59026582cf0d
Successfully tagged ubuntu:nginx
root@instance-20210604-1640:~#
```

RUN 명령은 아래 예시처럼 패키지 설치 등에 사용 되는건데, 좀 더 특정지어 말한다면 RUN 명령은 image layer를 만들어낸다.

```shell
RUN yum -y install nginx 
```

/bin/sh 또는 exec 형식으로 작성할 수 있는데 exec 형식은 JSON 배열로 지정된다.

```shell
RUN ["/bin/sh", "-c", "yum -y install nginx"]
```

RUN 명령은 되도록 한줄로 작성하는 것이 좋다.

images 명령으로 이미지를 확인하면 생성된 ubuntu:nginx 이미지를 확인할 수 있다.

```shell
root@instance-20210604-1640:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
ubuntu       nginx     59026582cf0d   8 minutes ago   161MB
ubuntu       latest    9873176a8ff5   11 days ago     72.7MB
```

### 컨테이너 실행시 실행할 명령(CMD와 ENTRYPOINT)

컨테이너 실행시 실행될 명령어를 지정해보자. Ubuntu에 nginx를 설치하였을 때 컨테이너를 실행하면 자동으로 nginx를 실행해야 한다. 그럴 때 사용하는 것이 CMD와 ENTRYPOINT이다. 그러나 Nginx를 잘 모르면 어려울 수 있기 때문에 다음에 다루기로 한다.

#### /bin/ls 실행하기

Dockerfile을 다음과 같이 작성한다.

```shell
FROM ubuntu:latest

CMD ["/bin/ls", "-al""]
```

빌드한다.

```shell
root@instance-20210604-1640:~# docker build --tag ubuntu:cmd .
Sending build context to Docker daemon  27.65kB
Step 1/2 : FROM ubuntu:latest
latest: Pulling from library/ubuntu
c549ccf8d472: Pull complete
Digest: sha256:aba80b77e27148d99c034a987e7da3a287ed455390352663418c0f2ed40417fe
Status: Downloaded newer image for ubuntu:latest
 ---> 9873176a8ff5
Step 2/2 : CMD ["/bin/ls", "-al"]
 ---> Running in 7871589f7ae0
Removing intermediate container 7871589f7ae0
 ---> 4fc750578532
Successfully built 4fc750578532
Successfully tagged ubuntu:cmd
```

일단 컨테이너를 실행해보자. ls -al이 실행되는 것을 볼 수 있다.

```shell
root@instance-20210604-1640:~# docker run ubuntu:cmd
total 56
drwxr-xr-x   1 root root 4096 Jun 29 01:54 .
drwxr-xr-x   1 root root 4096 Jun 29 01:54 ..
dr-xr-xr-x  13 root root    0 Jun 29 01:54 sys
drwxrwxrwt   2 root root 4096 Jun  9 07:31 tmp
drwxr-xr-x  13 root root 4096 Jun  9 07:27 usr
drwxr-xr-x  11 root root 4096 Jun  9 07:31 var
root@instance-20210604-1640:~#
```

#### 실행할 때 인자값 주기

컨테이너를 실행할 때 pwd 인자값을 준다.

```shell
docker run ubuntu:cmd pwd 
```

실행결과는 CMD의 명령이 실행되지 않고 다음과 같이 루트의 경로가 출력된다. docker run 명령에 쉘 명령어 및 인자값 전달할 경우 CMD에 작성된 명령어와 인자값은 무시된다는 것을 알 수 있다.

```shell
root@instance-20210604-1640:~# docker run ubuntu:cmd pwd
/
```

#### ENTRYPOINT

Dockefile을 다음과 같이 작성한다.

```shell
FROM ubuntu:latest

ENTRYPOINT ["/bin/ls", "-al"]
```

이미지를 빌드한다.

```shell
root@instance-20210604-1640:~# docker build --tag ubuntu:cmd2 .
Sending build context to Docker daemon  27.65kB
Step 1/2 : FROM ubuntu:latest
 ---> 9873176a8ff5
Step 2/2 : ENTRYPOINT ["/bin/ls", "-al"]
 ---> Running in e291a55a0ee6
Removing intermediate container e291a55a0ee6
 ---> 92b9e0e58df4
Successfully built 92b9e0e58df4
Successfully tagged ubuntu:cmd2
```

컨테이너를 실행한다. ls -al 명령이 실행된 것을 볼 수 있다.

```shell
root@instance-20210604-1640:~# docker run ubuntu:cmd2
total 56
drwxr-xr-x   1 root root 4096 Jun 29 02:02 .
drwxr-xr-x   1 root root 4096 Jun 29 02:02 ..
-rwxr-xr-x   1 root root    0 Jun 29 02:02 .dockerenv
lrwxrwxrwx   1 root root    7 Jun  9 07:27 bin -> usr/bin
drwxr-xr-x   2 root root 4096 Apr 15  2020 boot
drwxr-xr-x   5 root root  340 Jun 29 02:02 dev
drwxr-xr-x   1 root root 4096 Jun 29 02:02 etc
drwxr-xr-x   2 root root 4096 Apr 15  2020 home
```

이번에는 인자값을 주어 컨테이너를 실행한다.

```shell
docker run ubuntu:cmd2 -S
```

그러면 옵션이 적용된 것을 볼 수 있다. 결과가 너무 많아서 생략했다. ENTRYPOINT에 있는 ls는 실행이 되지만 -al 인자값은 -S로 변경된다.

### 복사하기(COPY/ADD)

#### 루트 경로의 파일 복사

다음과 같이 Dockefile을 생성한다.

```shell
FROM ubuntu:latest

COPY test.sh run.sh
RUN chmod 755 run.sh

ENTRYPOINT ["./run.sh"]
```

현재 경로에 test.sh를 만든다.

```shell
#!/bin/sh

echo "Hello World"
```

이미지를 빌드하고 컨테이너를 실행하면 "Hello World"가 출력되는 것을 확인할 수 있다.

```shell
root@instance-20210604-1640:~# docker run ubuntu:my-test
Hello World
```

COPY는 ADD와는 달리 압축 파일을 추가할 때 압축을 해제하지 않고 파일 URL도 사용할 수 없다.

COPY <복사할 파일경로> <이미제에서 파일이 위치할 경로>

* 복사할 파일의 경ㅇ로는 컨텍스트 아래를 기준으로 하며 바깥의 파일, 디렉터리나 절대 경로는 사용할 수 없다.
* 복사할 파일뿐만 아니라 디렉터리도 설정할 수 있다.
* 디렉터리를 지정하면 모든 파일을 복사한다.
* 와일득카드를 사용하여 특정 파일만 복사할 수 있다.

이미지에서 파일이 위치할 경로는 항상 절대 경로로 설정해야 한다. 그리고 마지막이 /로 끝나면 디렉터리가 생성되고 파일은 그 아래에 복사된다.

#### 특정 디렉터리에 복사

Dockerfile을 다음과 같이 생성한다.

```shell
FROM ubuntu:latest

COPY test.sh /root/run.sh
RUN chmod 755 /root/run.sh
ENTRYPOINT ["/root/run.sh"]
```

이미지를 빌드하고 컨테니너를 실행하면 'Hello World'를 출력할 것이다.

```shell
root@instance-20210604-1640:~# docker run ubuntu:cmd3
Hello World
```

#### ADD를 이용한 복사

COPY/ADD 명령 두개 다 호스트OS의 파일 또는 디렉토리를 컨테이너 안의 경로로 복사한다. COPY의 경우 호스트OS에서 컨테이너 안으로 복사만 가능하다. ADD의 경우 원격 파일 다운로드 또는 압축 해제 등과 같은 기능을 갖고 있다. 따라서, 호스트OS에서 컨테이너로 단순히 복사만을 처리할 때 COPY를 사용 한다.

ADD와 COPY 두 명령 사용법은 아래와 같고 소유자와 소유자 그룹도 수정이 가능하다. 더블 쿼트로 감싼 형식은 공백을 포함하는 경로일 경우에 사용한다.

```shell
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] 
```

```shell
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

ADD를 사용할 경우 URL을 입력하여 다운로드가 가능하다.

```shell
ADD http://my.github.com/index.html /root/html/index.html
```

로컬에 있는 압축 파일(tar.gz, tar.bz2, tar.xz)은 압축을 해제하고 tar를 풀어서 추가된다. 단, 인터넷에 있는 파일 URL은 압축만 해제한 뒤 tar 파일이 그대로 추가된다. 나머지는 COPY와 동일하다.

### 환경변수 설정(ENV/ARG)

ENV는 Dockerfile 또는 컨테이너 안에서 환경 변수로 사용이 가능하고, ARG는 Dockerfile 에서만 사용이 가능하다.

다음과 같이 Dockerfile을 생성한다.

```shell
FROM ubuntu:latest

ENV MY_NAME LATTE
ENV MY_EMAIL LATTE@GMAIL.COM

RUN echo ${MY_NAME}
RUN echo ${MY_EMAIL}
```

이미지를 빌드하면 환경 변수 결과가 출력되는 것을 볼 수 있다.

```shell
# docker build --tag ubuntu:cmd4 .
Sending build context to Docker daemon  29.18kB
Step 1/5 : FROM ubuntu:latest
 ---> 9873176a8ff5
Step 2/5 : ENV MY_NAME LATTE
 ---> Running in f0ffb236a244
Removing intermediate container f0ffb236a244
 ---> 786a7c36ef91
Step 3/5 : ENV MY_EMAIL LATTE@GMAIL.COM
 ---> Running in ef6c119334a3
Removing intermediate container ef6c119334a3
 ---> ae6ca15ff748
Step 4/5 : RUN echo ${MY_NAME}
 ---> Running in 3ac6c1c6772d
LATTE
Removing intermediate container 3ac6c1c6772d
 ---> 811fd916256c
Step 5/5 : RUN echo ${MY_EMAIL}
 ---> Running in 9a8426f9976b
LATTE@GMAIL.COM
Removing intermediate container 9a8426f9976b
 ---> 7b3ec47ecc65
```

그러나 컨테이너를 실행하면 결과가 출력되지 않는다. RUN을 사용했기 때문이다. test.sh 파일을 다음과 같이 생성한다.

```shell
#!/bin/sh

echo $MY_NAME
echo $MY_EMAIL
```

Dockefile을 다음과 같이 작성한다.

```shell
FROM ubuntu:latest

ENV MY_NAME LATTE
ENV MY_EMAIL LATTE@GMAIL.COM

COPY test.sh run.sh
RUN chmod 755 run.sh
ENTRYPOINT ["./run.sh"]
```

컨테이너를 실행하면 환경 변수 값이 출력된느 것을 볼 수 있다.

```shell
root@instance-20210604-1640:~# docker run ubuntu:cmd6
LATTE
LATTE@GMAIL.COM
root@instance-2021
```

> Dockefile 자체에서만 사용할 변수는 ARG를 사용한다.

### WORKDIR

WORKDIR은 명령을 실행하기 위한 디렉토리를 지정한다. 먼저 실행할 명령어 파일을 작성한다.

```shell
#!/bin/sh
pwd
echo "Hello World"
```

Dockerfile을 다음과 같이 작성한다. 명령어 파일을 /root/cont/에 복사한다. 컨테이너를 실행하면 WORKDIR로 이동한다. 그리고 CMD를 실행하게 된다.

```shell
FROM ubuntu:latest

COPY run.sh /root/cont/run.sh
WORKDIR /root/cont/

CMD ./run.sh
```

### VOLUME

컨테이너 안에 있는 데이터는 컨테이너를 삭제하면 모든 데이터가 같이 삭제(휘발성 데이터) 된다. 따라서 데이터를 보존하기 위해 VOLUME을 사용 하고, VOLUME 명령은 설정한 컨테이너의 데이터를 호스트 OS에 저장하거나, 컨테이너들간의 데이터를 공유가 가능하다.

```shell
VOLUME ["디렉터리1", "디렉터리2"]
```

Dockerfile 에서 위와 같이 생성한 볼륨은 호스트OS의 '/var/lib/docker/volumes'에 생성된다. Docker 에서 자동 생성한 hash값으로 디렉토리가 생성된다.

### USER

USER 명령은 RUN, CMD, ENTRYPOINT와 같은 명령을 실행하기 위한 특정 사용자를 지정해야 하는 상황에서 사용된다. 아래와 같이 유저명:그륩명 또는 UID:GID 와 같이 사용 되고, 그륩명과 GID는 생략이 가능하다. 단, 사용 시 주의점이 있는데 USER 명령을 사용하기 위한 사용자 계정이 존재 해야 한다. USER 명령은 계정을 생성하는것이 아니라, 특정 사용자 계정을 사용하기 위한 명령이다.

```shell
USER <user>[:<group>]
or
USER <UID>[:<GID>]
```

useradd \[사용자명] 을 미리 입력하여 계정을 생성하고, 리눅스의 id 명령어로 확인할 수 있다.

```shell
RUN useradd latte  # root 계정으로 실행
RUN id

USER latte
RUN id  # latte 계정으로 실행
```

### LABEL

LABEL 명령은 이미지의 버전 정보, 작성자 등 이미지의 상세 정보를 작성해 두기 위한 용도이다.

```shell
LABEL title="webserver"
LABEL version="1.0"
```

### EXPOSE

EXPOSE 명령은 해당 컨테이너가 런타임에 지정된 네트워크 포트에서 수신 대기중 이라는것을 알려준다. dockerfile을 작성하는 사람과 컨테이너를 직접 실행할 사람 사이에서 공개할 포트를 알려주기 위해 문서 유형으로 작성할 때 사용된다.

```shell
EXPOSE 80/tcp
EXPOSE 80/udp 
```

### HEALTHCHECK

컨테이너의 HEALTHCHECK를 사용하여 컨테이너의 프로세스 상태를 체크할 수 있는데 두가지 사용 방법이 있다.

* **HEALTHCHECK \[OPTIONS] CMD command**\
  컨테이너 내부에서 명령 실행하여 컨테이너 상태 확인, 이 방법을 통해 웹페이지 등을 확인할 수 있다.
* **HEALTHCHECK NONE** (베이스 이미지에서 상속된 상태 확인을 비활성화) 또한 Dockerfile에서 HEALTHCHECK는 하나의 명령만이 유효하고, 만약 여러개가 있다면 가장 마지막에 선언된 HEALTHCHECK가 적용 된다.

| 옵션                  | 설명       | 기본값 |
| ------------------- | -------- | --- |
| --interval=DURATION | 헬스 체크 간격 | 30s |
| --timeout=DURATION  | 타임 아웃    | 30s |
| --retries=N         | 타임 아웃 횟수 | 3   |

HEALTHCHECK의 처음 상태는 starting 이고, HEALTHCHECK가 통과될 때 마다 healthy (이전 상태와 상관없이) 됩니다. 그리고 옵션에 정한 일정 횟수가 실패된다면 unhealthy 상태로 됩니다

| EXIT CODE    | 설명                    |
| ------------ | --------------------- |
| 0: success   | 컨테이너가 정상적이고 사용 가능한 상태 |
| 1: unhealthy | 컨테이너가 올바르게 작동하지 않는 상태 |
| 2: starting  | 예약된 코드                |

다음은 HEALTHCHCK를 위한 설정의 예이다.

```shell
HEALTHCHECK --interval=10s --timeout=3s CMD curl -f http://127.0.0.1/ || exit 1
```

* \--interval=10s는 10초마다 healthcheck를 실행한다.
* \--timout=3s는 3초 이상이 소요되면 3번의 재시도를 한다.(retries 초기값은 3)
* 실패하면 unhealty 상태로 변경된다.

### STOPSIGNAL

docker container stop 명령을 입력하면 Docker 데몬이 컨테이너에게 signal을 보내 중지 하는데, 기본적으로 STOPSIGNAL을 명시하지 않을 경우 SIGTERM을 사용 하게된다.

```shell
STOPSIGNAL [시그널]   
```

docker container stop 실행 시 SIGTERM signal을 받은 컨테이너가 프로세스를 정상적으로 종료할 수 있을때 까지 기다리게 되는데, 지정된 시간 (deafult 10sec, 사용자 지정 가능)동안 종료가 되지 않으면 SIGKILL을 전송한다. 'kill -l'로 정의된 시그널들을 볼 수 있다.

```shell

root@instance-20210604-1640:~# kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

### ONBUILD

ONBULD는 조금 특이하게 처음 사용한 Dockerfeil 에서 빌드할 때(이미지 생성) 실행되는 명령이 아니다. ONBUILD 명령을 사용했던 이미지를, 다른 Dockerfile에서 FROM image를 사용하여 빌드 했을 때 동작한다.
