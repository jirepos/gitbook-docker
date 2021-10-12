# Docker Image 이해

## 요약

### 이미지 받기

최신 버전의 nginx 이미지 받기

```shell
docker pull nginx:latest
```

6 개의 레이어가 받아진다.

```shell
root@instance-20210604-1640:~# docker pull nginx:latest
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

### 도커 기본 데이터 저장 경로

```shell
/var/lib/docker
```

Overlay2 드라이버로 저장된 데이터 경로

```shell
/var/lib/docker/image/overlay2/layerdb/sha256
```

이동한다.

```shell
cd /var/lib/docker/image/overlay2/layerdb/sha256
```

6개의 디렉터리가 있는 것을 확인한다. 레이어들은 독립적으로 저장된다. docker pull로 받을 때 출력되는 다이제스트 값과는 다르다. pull로 받을 때 출력된느 값은 도커 레지스트리가 관리하는 값이고 실제 레이어 아이디 값을 다르다.

```shell

root@instance-20210604-1640:/var/lib/docker/image/overlay2/layerdb/sha256# ls -al
total 32
drwxr-xr-x 8 root root 4096 Jun 28 22:55 .
drwx------ 5 root root 4096 Jun 28 03:25 ..
drwx------ 2 root root 4096 Jun 28 22:55 038ca5b801cea48e9f40f6ffb4cda61a2fe0b6b0f378a7434a0d39d2575a4082
drwx------ 2 root root 4096 Jun 28 22:55 2855bbcefcf95050e64049447e99e77efa2bff32374e586982d69be4612467ce
drwx------ 2 root root 4096 Jun 28 22:55 36d83ebf5fec7ae1be4c431f0945f2dbe6828ecdc936c604daa48f17c0b50ed7
drwx------ 2 root root 4096 Jun 28 22:54 764055ebc9a7a290b64d17cf9ea550f1099c202d83795aa967428ebdf335c9f7
drwx------ 2 root root 4096 Jun 28 22:55 b4c9a251dc81d52dd1cca9b4c69ca9e4db602a9a7974019f212846577f739699
drwx------ 2 root root 4096 Jun 28 22:55 bad169ad8b30eab551acbb8cd8fbdcd824528189e3dd0cc52dd88a37bbf121cd
```

### 도커 레이어목록 확인

docker image inspect 명령으로 레이어 목록을 확인할 수 있다.

```shell
docker inspect ubuntu:latest 
```

다음과 같은 결과가 출력된다. "/var/lib/docker/image/overlay2/layerdb/sha256" 디렉터리에 있는 디렉터리 이름들과 같다.

```shell
"RootFS": {
  "Type": "layers",
     "Layers": [
        "sha256:764055ebc9a7a290b64d17cf9ea550f1099c202d83795aa967428ebdf335c9f7",
        "sha256:2418679ca01f484b28bdcd8606d1d5313013cccfbd395123716000c2a25eec09",
        "sha256:cf388fcf3527352baa4ee5bedafd223b2224b13aac5f2df1ea0be47422789892",
        "sha256:165eb6c3c0d39f8c99de581f1bf9cc094e7823a838fbe5c51574c7beb3f6c4ee",
        "sha256:b50a193ebf2e2579b49d30beb9798078e22db0d4490bf51b9b3bb0d8bb7a3833",
        "sha256:c6d74dcb7fe747fac8e74e9453156380ea3ffddf97ed0e6c71e75400d938216e"
        ]
},
```

### 실제레이어 내용 확인

레이어 디렉터리를 이동한다.

```shell
cd 764055ebc9a7a290b64d17cf9ea550f1099c202d83795aa967428ebdf335c9f7
```

디렉터리의 파일 목록을 확인한다.

```shell
# ls -al
total 264
drwx------ 2 root root   4096 Jun 28 22:54 .
drwxr-xr-x 8 root root   4096 Jun 28 22:55 ..
-rw-r--r-- 1 root root     64 Jun 28 22:54 cache-id
-rw-r--r-- 1 root root     71 Jun 28 22:54 diff
-rw-r--r-- 1 root root      8 Jun 28 22:54 size
-rw-r--r-- 1 root root 249737 Jun 28 22:54 tar-split.json.gz
```

cache-id라는 파일의 실제 값을 출력하면 실제 디렉터리를 확인할 수 있다.

```shell
# cat cache-id; echo
```

```shell
# cat cache-id; echo
bd3b1d29225debaf1681eee25c5fb167541d805a0c51d79bb755567264584be4
```

이 디렉터리는 다음의 경로 바로 아래에 있다.

```shell
/var/lib/docker/overlay2
```

실제 디렉터리로 이동한다.

```shell
cd /var/lib/docker/overlay2/bd3b1d29225debaf1681eee25c5fb167541d805a0c51d79bb755567264584be4
```

파일 목록을 확인한다.

```shell
# ls -al
total 16
drwx-----x  3 root root 4096 Jun 28 22:54 .
drwx-----x  9 root root 4096 Jun 28 22:55 ..
-rw-------  1 root root    0 Jun 28 22:54 committed
drwxr-xr-x 21 root root 4096 Jun 28 22:54 diff
-rw-r--r--  1 root root   26 Jun 28 22:54 link
```

실제 컨텐츠는 diff 폴더에 들어있다. 이동한다.

```shell
cd diff
```

목록을 출력한다.

```shell
# ls -al
drwxr-xr-x 21 root root 4096 Jun 28 22:54 .
drwx-----x  3 root root 4096 Jun 28 22:54 ..
drwxr-xr-x  2 root root 4096 Jun 21 00:00 bin
drwxr-xr-x  2 root root 4096 Jun 13 10:30 boot
drwxr-xr-x  2 root root 4096 Jun 21 00:00 dev
drwxr-xr-x 28 root root 4096 Jun 21 00:00 etc
drwxr-xr-x  2 root root 4096 Jun 13 10:30 home
drwxr-xr-x  7 root root 4096 Jun 21 00:00 lib
drwxr-xr-x  2 root root 4096 Jun 21 00:00 lib64
drwxr-xr-x  2 root root 4096 Jun 21 00:00 media
drwxr-xr-x  2 root root 4096 Jun 21 00:00 mnt
drwxr-xr-x  2 root root 4096 Jun 21 00:00 opt
drwxr-xr-x  2 root root 4096 Jun 13 10:30 proc
drwx------  2 root root 4096 Jun 21 00:00 root
drwxr-xr-x  3 root root 4096 Jun 21 00:00 run
drwxr-xr-x  2 root root 4096 Jun 21 00:00 sbin
drwxr-xr-x  2 root root 4096 Jun 21 00:00 srv
drwxr-xr-x  2 root root 4096 Jun 13 10:30 sys
drwxrwxrwt  2 root root 4096 Jun 21 00:00 tmp
drwxr-xr-x 10 root root 4096 Jun 21 00:00 usr
drwxr-xr-x 11 root root 4096 Jun 21 00:00 var
```

### 이미지 레이어 확인

history 명령으로 이미지의 레이어를 확인한다.

```shell
docker history ubuntu:latest
```

```shell
# docker history nginx:latest
IMAGE          CREATED      CREATED BY                                      SIZE      COMMENT
4f380adfc10f   5 days ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>      5 days ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
<missing>      5 days ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>      5 days ago   /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
<missing>      5 days ago   /bin/sh -c #(nop) COPY file:09a214a3e07c919a…   4.61kB
<missing>      5 days ago   /bin/sh -c #(nop) COPY file:0fd5fca330dcd6a7…   1.04kB
<missing>      5 days ago   /bin/sh -c #(nop) COPY file:0b866ff3fc1ef5b0…   1.96kB
<missing>      5 days ago   /bin/sh -c #(nop) COPY file:65504f71f5855ca0…   1.2kB
<missing>      5 days ago   /bin/sh -c set -x     && addgroup --system -…   63.8MB
<missing>      5 days ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster     0B
<missing>      5 days ago   /bin/sh -c #(nop)  ENV NJS_VERSION=0.5.3        0B
<missing>      5 days ago   /bin/sh -c #(nop)  ENV NGINX_VERSION=1.21.0     0B
<missing>      5 days ago   /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
<missing>      5 days ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      5 days ago   /bin/sh -c #(nop) ADD file:4903a19c327468b0e…   69.2MB
```

맨 위줄의 4f380adfc10f 아이디가 docker images의 아이디와 일치한다.

```shell
# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
nginx        latest    4f380adfc10f   5 days ago   133MB
```

### 컨테이너 실행

```shell
docker run -it nginx:latest bash
root@1476530df65f:/#
```

컨테이너를 실행하면 아주 깨끗한 레이어를 최상위 위의 레이어에 쌓아 올린다. 방금 생성한 컨테이너의 아이디는 '1476530df65f'이다.

### 컨테이너 마운트 구조 확인

```shell
docker inspect 1476530df65f
```

```shell
"GraphDriver": {
  "Data": {
    "LowerDir": "/var/lib/docker/overlay2/3eb60e4e2d659d0e37f790bfa2905ee4008e57e23750bd6ad35c016bd4d14c7c-init/diff:/var/lib/docker/overlay2/bca1fe8ba67dda9684b9115d99003355b382bc289a328371a5410f783853312a/diff:/var/lib/docker/overlay2/671d9bd4f00e87b987fe81e1507065a63301fc486f1149ae41b543c6fcdb7a79/diff:/var/lib/docker/overlay2/f4f2440de54f5e213015115dec0f9920e5289eddc19e01136b3a9fbf74b1c332/diff:/var/lib/docker/overlay2/fb82a6cc1c96993285a4494eb34a74537b9e347e36e84341169a6c7d6b8ee341/diff:/var/lib/docker/overlay2/d3673db21018e06d393a43e367fb2a77f4698a5e0fe6b50c515084d4162799db/diff:/var/lib/docker/overlay2/bd3b1d29225debaf1681eee25c5fb167541d805a0c51d79bb755567264584be4/diff",
    "MergedDir": "/var/lib/docker/overlay2/3eb60e4e2d659d0e37f790bfa2905ee4008e57e23750bd6ad35c016bd4d14c7c/merged",
    "UpperDir": "/var/lib/docker/overlay2/3eb60e4e2d659d0e37f790bfa2905ee4008e57e23750bd6ad35c016bd4d14c7c/diff",
    "WorkDir": "/var/lib/docker/overlay2/3eb60e4e2d659d0e37f790bfa2905ee4008e57e23750bd6ad35c016bd4d14c7c/work"
},
```

* LowerDir: 이미지 레이어들
* UpperDir: 컨테이너 레이어

UpperDir로 이동한다. 목록을 확인한다.

```shell
ls -al
```

UpperDir에 설정된 '3eb60e4e2d659d0e37f790bfa2905ee4008e57e23750bd6ad35c016bd4d14c7c' 디렉터리가 존재하는 것을 볼 수 있다.

```shell
root@instance-20210604-1640:/var/lib/docker/overlay2# ls -al
total 44
drwx-----x 11 root root 4096 Jun 28 23:47 .
drwx--x--x 13 root root 4096 Jun 24 10:31 ..
drwx-----x  5 root root 4096 Jun 28 23:47 3eb60e4e2d659d0e37f790bfa2905ee4008e57e23750bd6ad35c016bd4d14c7c
drwx-----x  4 root root 4096 Jun 28 23:47 3eb60e4e2d659d0e37f790bfa2905ee4008e57e23750bd6ad35c016bd4d14c7c-init
drwx-----x  4 root root 4096 Jun 28 22:55 671d9bd4f00e87b987fe81e1507065a63301fc486f1149ae41b543c6fcdb7a79
drwx-----x  4 root root 4096 Jun 28 23:47 bca1fe8ba67dda9684b9115d99003355b382bc289a328371a5410f783853312a
drwx-----x  3 root root 4096 Jun 28 22:54 bd3b1d29225debaf1681eee25c5fb167541d805a0c51d79bb755567264584be4
drwx-----x  4 root root 4096 Jun 28 22:55 d3673db21018e06d393a43e367fb2a77f4698a5e0fe6b50c515084d4162799db
drwx-----x  4 root root 4096 Jun 28 22:55 f4f2440de54f5e213015115dec0f9920e5289eddc19e01136b3a9fbf74b1c332
drwx-----x  4 root root 4096 Jun 28 22:55 fb82a6cc1c96993285a4494eb34a74537b9e347e36e84341169a6c7d6b8ee341
```

이 디렉터리의 diff로 이동한다.

```shell
cd 3eb60e4e2d659d0e37f790bfa2905ee4008e57e23750bd6ad35c016bd4d14c7c/diff
```

목록을 출력하면 아무것도 없다.

```shell
ls -al
```

### 컨테이너 변경 사항 확인

레이어에 변경된 사항을 확인하기 위햇 diff 명령을 사용한다. 역시 아무것도 하지 않았기 때문에 아무것도 출력이 안된다.

```shell
docker diff 1476530df65f
```

### 컨테이너에 파일 생성

#### 파일 생성

tmp 디렉터리로 이동해서 파일을 하나 생성한다.

```shell
cd tmp
touch THIS_IS_CONTAINER
```

### 컨테이너 변경사항 확인

다시 diff 명령을 실행한다.

```shell
docker diff 1476530df65f
C /tmp
A /tmp/THIS_IS_CONTAINER
```

이 번에는 변경사항이 출력된다. 실제 레이어 디렉터리(UPPERDIR)로 이동하여 변경사항을 확인한다.

```shell
cd /var/lib/docker/overlay2/3eb60e4e2d659d0e37f790bfa2905ee4008e57e23750bd6ad35c016bd4d14c7c/diff
```

다음과 같이 변경사항이 출력된다.

```shell
# ls -al
drwxr-xr-x 3 root root 4096 Jun 28 23:47 .
drwx-----x 5 root root 4096 Jun 28 23:47 ..
drwxrwxrwt 2 root root 4096 Jun 29 00:00 tmp
```

#### 컨테이너 레이어 디렉터리 확인

tmp로 들어가서 보면 파일이 보일 것이다.

```shell
#cd tmp
# ls
THIS_IS_CONTAINER
```

tree 명령으로 확인할 수 있다.

```shell
# tree
.
└── tmp
    └── THIS_IS_CONTAINER
1 directory, 1 file
```

### 컨테이너 파일 삭제

#### 파일 삭제

이미지에 저장되어 있는 tar 파일을 삭제한다.

```shell
rm /bin/tar
```

#### 변경사항 확인

diff로 변경사항을 확인한다. 이미지 레이어에 있는 파일을 삭제하더라도 컨테이너 레이어에 기록된다.

```shell
docker diff 1476530df65f
```

결과는 다음과 같다.

```shell
#  docker diff 1476530df65f
C /bin
D /bin/tar
C /tmp
A /tmp/THIS_IS_CONTAINER
```

#### 컨테이너 레이어 디렉터리 확인

tree 명령으로 컨테이너 레이어의 변경사항을 확인한다. 색깔이 다르게 표시된느데 OverlayFS에서는 하위 레이어의 파일이 존재하지 않는 것처럼 가리는 역할을 한다.

```shell
# tree
.
├── bin
│   └── tar
└── tmp
    └── THIS_IS_CONTAINER
```

실제로는 원래의 이미지의 레이어에서는 파일이 삭제되지 않는다. 읽기 전용이라는 것을 기억해야 한다.

### 새로운 이미지 생성

#### 이미지 생성

docker commit 명령으로 새로운 이미지를 생성한다.

```shell
docker commit 1476530df65f nginx:my-test
```

생성된 이미지의 다이제스트 값이 출력된다.

```shell
sha256:8d94a90d5cb799f7c6940088227741e72d16b4d82e9fe72a6c4ed13c12854697
```

이 이미지를 컨테이너를 실행하면 tar를 사용할 수 없다.

```shell
docker run -it nginx:my-test bash 
# tar 
base: tar: command not found
#
```

#### 이미지 확인

```shell
docker hisory nginx:my-test 
```

nginx:latest 이미지에 새로운 레이어가 올라 간 것을 볼 수 있다.

```shell
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
8d94a90d5cb7   2 minutes ago   bash                                            0B
4f380adfc10f   5 days ago      /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>      5 days ago      /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
<missing>      5 days ago      /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>      5 days ago      /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
<missing>      5 days ago      /bin/sh -c #(nop) COPY file:09a214a3e07c919a…   4.61kB
<missing>      5 days ago      /bin/sh -c #(nop) COPY file:0fd5fca330dcd6a7…   1.04kB
<missing>      5 days ago      /bin/sh -c #(nop) COPY file:0b866ff3fc1ef5b0…   1.96kB
<missing>      5 days ago      /bin/sh -c #(nop) COPY file:65504f71f5855ca0…   1.2kB
<missing>      5 days ago      /bin/sh -c set -x     && addgroup --system -…   63.8MB
<missing>      5 days ago      /bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster     0B
<missing>      5 days ago      /bin/sh -c #(nop)  ENV NJS_VERSION=0.5.3        0B
<missing>      5 days ago      /bin/sh -c #(nop)  ENV NGINX_VERSION=1.21.0     0B
<missing>      5 days ago      /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
<missing>      6 days ago      /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      6 days ago      /bin/sh -c #(nop) ADD file:4903a19c327468b0e…   69.2MB
```

이 글은 아래 글을 따라하면서 정리한 글이다. 좀 더 자세한 설명은 아래 사이트를 참고한다. [https://www.44bits.io/ko/post/how-docker-image-work](https://www.44bits.io/ko/post/how-docker-image-work)

## Docker 이미지 받기

'pull'을 사용하여 Nginx 최신 버전을 받아 보자.

```shell
$ sudo docker pull nginx:latest
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

이미지를 받는 과정에서 출력되는 내용에는 여러가지 정보가 포함되어 있다. 마지막 줄의 내용을 살펴 보겠다.

```
docker.io/library/nginx:latest
```

도커 허브를 기준으로 도커 이미지 이름은 \<NAMESPADCE>/\<IMAGE_NAME>:\<TAG>로 구성이 된다.

```shell
<NAMESPADCE>/<IMAGE_NAME>:<TAG>
```

library는 도커 허브의 공식 이미지가 저장되어있는 특별한 네임스페이스이다. 보통은 사용자의 이름이 온다. 슬래시로 구분된 맨 앞에는 도커 이미지 저장소의 실제 주소를 나타낸다.

docker info 명령을 사용하여 정보를 출력해보면 아래 쪽에 Registry 항목이 나온다. 기본 도커 저장소의 주소인데 다른 주소이긴 하지만 같은 실제로는 같은 주소를 가르킨다.

```shell
docker info
```

```shell
root> docker info
...
Name: instance-20210604-1640
 ID: D7KC:RRZW:S7KM:2D55:3HNP:4NJO:CDE7:3IYN:BZJO:NO6P:NSCE:VNFF
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
```

이미지를 받을 때 Digest 항목이 출력되는 것을 볼 수 있다. 이 값을 가지고 이미지를 받을 수도 있다.

```shell
Digest: sha256:47ae43cdfc7064d28800bc42e79a429540c7c80168e8c8952778c0d5af1c09db
```

## Docker 이미지 저장 경로

이미지를 받을 때 콘솔에 출력되는 내용을 보면 무엇인가를 나누어서 받고 있다. 도커의 이미지는 레이어들로 구성이 되는데 이 레이어들은 독립적으로 저장이 된다. 컨테이너를 실행할 때에는 이 레이어들을 차례로 쌓아 올려서 특정 위치에 마운트 한다. 아래 그림은 도커 이미지의 레이어 구조를 보여준다.

```shell
--------------------
Layer 3
--------------------
Layer 2
--------------------
Layer 1
--------------------
```

이 레이어들은 읽기 전용이다. 마지막으로 쓰기 가능한 레이어를 한 층 더 올리고, 컨테이너에서 일어나는 모든 변경 사항을 이 레이어에 저장한다.

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

### 도커 기본 데이터 저장 경로

도커의 기본 데이터는 기본적으로 다음의 경로에 저장된다.

```shell
/var/lib/docker
```

Overlay2 드라이버로 저장된 레이어 데이터는 다시 다시 image/overlay2/layerdb/sha256 아래에 저장된다.

```shell
root> pwd
/var/lib/docker/image/overlay2/layerdb/sha256
root> ls -al
total 32
drwxr-xr-x 8 root root 4096 Jun 28 02:02 .
drwx------ 4 root root 4096 Jun 28 02:01 ..
drwx------ 2 root root 4096 Jun 28 02:02 038ca5b801cea48e9f40f6ffb4cda61a2fe0b6b0f378a7434a0d39d2575a4082
drwx------ 2 root root 4096 Jun 28 02:02 2855bbcefcf95050e64049447e99e77efa2bff32374e586982d69be4612467ce
drwx------ 2 root root 4096 Jun 28 02:02 36d83ebf5fec7ae1be4c431f0945f2dbe6828ecdc936c604daa48f17c0b50ed7
drwx------ 2 root root 4096 Jun 28 02:01 764055ebc9a7a290b64d17cf9ea550f1099c202d83795aa967428ebdf335c9f7
drwx------ 2 root root 4096 Jun 28 02:02 b4c9a251dc81d52dd1cca9b4c69ca9e4db602a9a7974019f212846577f739699
drwx------ 2 root root 4096 Jun 28 02:02 bad169ad8b30eab551acbb8cd8fbdcd824528189e3dd0cc52dd88a37bbf121cd
```

> **Union Mount File System** 유니온 마운트란, 하나의 디렉터리 지점에 여러 개의 디렉터리를 마운트함으로써, 마치 하나의 통합된 디렉터리처럼 보이게 하는 것을 의미한다. 이를 간단하게 설명하자면, 마치 투명한 셀로판지를 여러 개 겹쳐서 위에서 내려다 보는 것과 비슷하다고 할 수 있다. 겹쳐진 여러 개의 셀로판지를 위에서 보게 되면 가장 위쪽에 있는 셀로판지의 그림만이 보일 것이고, 아래의 셀로판지 그림은 위쪽에 위치한 셀로판지에 의해 가려져서 보이지 않기 때문이다.

> **Overlay FS** 유니온 마운트를 지원하는 파일 시스템으로는 AUFS, OverlayFS 등이 있다. 도커에서 OverlayFS를 사용한다. 도커 이미지의 각 레이어가 lower 레이어에 해당하며, 새롭게 생성된 컨테이너의 컨테이너 레이어가 upper 레이어에 해당한다. 도커 이미지는 읽기 전용으로 불변의 상태를 유지해야 하므로, 도커 이미지에서 변경된 사항들은 R/W 전용의 컨테이너 레이어에 디렉터리에 저장된다. 그러나 사용자가 실제로 컨테이너 내부에서 파일 시스템을 사용할 때는 컨테이너 레이어 (upper) 와 이미지 레이어 (lower) 를 통합함으로써 완전한 디렉터리 (merge) 를 보여주게 된다

그런데 도커 이미지를 받을 때 출력된 아이디와 저장된 레이어의 아이다와 다르다.**이미지를 가져올 때 출력되는 다이제스트 값은 도커 레지스트리에서 관리하는 고유 아이디**이다. 레이어의 아이디는 별개로 있다. **docker image inspect** 명령으로 레이어 목록을 확인할 수 있다. RootFS의 Layers에 보면 sha256 디렉토리 아래의 디렉토리 이름과 같다는 것을 알 수 있다.

```shell
"RootFS": {
    "Type": "layers",
    "Layers": [
        "sha256:764055ebc9a7a290b64d17cf9ea550f1099c202d83795aa967428ebdf335c9f7",
        "sha256:2418679ca01f484b28bdcd8606d1d5313013cccfbd395123716000c2a25eec09",
        "sha256:cf388fcf3527352baa4ee5bedafd223b2224b13aac5f2df1ea0be47422789892",
        "sha256:165eb6c3c0d39f8c99de581f1bf9cc094e7823a838fbe5c51574c7beb3f6c4ee",
        "sha256:b50a193ebf2e2579b49d30beb9798078e22db0d4490bf51b9b3bb0d8bb7a3833",
        "sha256:c6d74dcb7fe747fac8e74e9453156380ea3ffddf97ed0e6c71e75400d938216e"
    ]
},
```

실제 레이어의 한 디렉터리로 들어가서 디디렉터리의 내용을 확인해 보자.

```shell
root> cd /var/lib/docker/image/overlay2/layerdb/sha256
root> cd 764055ebc9a7a290b64d17cf9ea550f1099c202d83795aa967428ebdf335c9f7
root> ls -al
total 264
drwx------ 2 root root   4096 Jun 28 02:01 .
drwxr-xr-x 8 root root   4096 Jun 28 02:02 ..
-rw-r--r-- 1 root root     64 Jun 28 02:01 cache-id
-rw-r--r-- 1 root root     71 Jun 28 02:01 diff
-rw-r--r-- 1 root root      8 Jun 28 02:01 size
-rw-r--r-- 1 root root 249737 Jun 28 02:01 tar-split.json.gz
```

여기에는 실제 레이어에 포함된 파일이 있지는 않다. 실제 데이터는 또 다른 곳에 있다. **cache-id**라는 파일이 있는데 이 값을 출력해 보면 실제 데이터가 있는 디렉터리의 다이제스트 값이 출력된다. 레이어 ID는 고유한 값이지만 cache-id는 이미지를 받는 시스템 마다 달라진다.

### 실제 레이어 데이터 경로 확인

```shell
root> cat cache-id
796eb70633c9cfc1f0c786b47d43304419b55cde09b47fe826eac5474203aec3root@instance-20210604-1640:/var/lib/docker/image/overlay2/layerdb/sha256/764055ebc9a7a290b64d17cf9ea550f1099c202d83795aa967428ebdf335c9f7
```

실제 데이터는 /var/lib/docker/overlay2 디렉터리에 있다. 디렉터리로 이동한다.

```shell
root> cd /var/lib/docker/overlay2
root> cd 796eb70633c9cfc1f0c786b47d43304419b55cde09b47fe826eac5474203aec3
```

그 디렉토리의 목록을 출력한다.

```shell
root@instance-20210604-1640:/var/lib/docker/overlay2/796eb70633c9cfc1f0c786b47d43304419b55cde09b47fe826eac5474203aec3# ls -al
total 16
drwx-----x  3 root root 4096 Jun 28 02:01 .
drwx-----x  9 root root 4096 Jun 28 02:02 ..
-rw-------  1 root root    0 Jun 28 02:01 committed
drwxr-xr-x 21 root root 4096 Jun 28 02:01 diff
-rw-r--r--  1 root root   26 Jun 28 02:01 link
```

### 실제 데이터 디렉터리(diff)

diff로 들어간다.

```shell
cd diff
```

diff 디렉터리의 목록을 출력한다. diff 디렉터리에 레이어의 컨텐츠가 있는 것을 볼 수 있다.

```shell
root> ls -al
total 84
drwxr-xr-x 21 root root 4096 Jun 28 02:01 .
drwx-----x  3 root root 4096 Jun 28 02:01 ..
drwxr-xr-x  2 root root 4096 Jun 21 00:00 bin
drwxr-xr-x  2 root root 4096 Jun 13 10:30 boot
drwxr-xr-x  2 root root 4096 Jun 21 00:00 dev
drwxr-xr-x 28 root root 4096 Jun 21 00:00 etc
drwxr-xr-x  2 root root 4096 Jun 13 10:30 home
drwxr-xr-x  7 root root 4096 Jun 21 00:00 lib
drwxr-xr-x  2 root root 4096 Jun 21 00:00 lib64
drwxr-xr-x  2 root root 4096 Jun 21 00:00 media
drwxr-xr-x  2 root root 4096 Jun 21 00:00 mnt
drwxr-xr-x  2 root root 4096 Jun 21 00:00 opt
drwxr-xr-x  2 root root 4096 Jun 13 10:30 proc
drwx------  2 root root 4096 Jun 21 00:00 root
drwxr-xr-x  3 root root 4096 Jun 21 00:00 run
drwxr-xr-x  2 root root 4096 Jun 21 00:00 sbin
drwxr-xr-x  2 root root 4096 Jun 21 00:00 srv
drwxr-xr-x  2 root root 4096 Jun 13 10:30 sys
drwxrwxrwt  2 root root 4096 Jun 21 00:00 tmp
drwxr-xr-x 10 root root 4096 Jun 21 00:00 usr
drwxr-xr-x 11 root root 4096 Jun 21 00:00 var
```

## 컨테이너 레이어 계층 이해

레이어는 아래에서부터 위로 쌓아올라간다. 따라서 layer1이 바닥에 있고, 그 위에 layer2가 올라가고 마지막으로 layer3가 위에 올라가서 비로소 이미지가 된다. 이미지의 레이어들은 모든 읽기 전용이다. 컨테이너에서 무엇을 하더라도 절대로 이미지의 내용이 달라지지 않는다. 컨테이너를 실행할 때 아주 깨끗한 레이어를 하나 이미지의 최상위 레이어 위에 올린다.

```shell
root> docker run -it nginx:latest bash
root@14f80d300a86:/# 
```

방금 생성한 컨테이너 아이디(호스트 이름)은 '14f80d300a86'이다. 임으로 이름을 붙여 container-layer-14f80d300a86라고 이름을 붙여 보겠다. 따라서 컨테이너 실행 시에 마운트되는 구조는 다음과 같을 것이다.

```shell
container-layer-14f80d300a86
layer 3 - nginx:latest
layer 2
layer 1
```

container-layer-14f80d300a86 레이어에는 아무 것도 없다. 하지만 컨테이너에서 파일 목록을 확인해 보면 그 아래 레이어에 있는 내용들이 그대로 보일 것이다. 컨테이너 레이어 아래에 있는 레이어들은 어떠한 변경도 일어나지 않는다. 아래 레이어에 있는 파일을 삭제한다면 최상위 레이어에 기록되고 그 아래에 있는 레이어들에는 아무런 변화가 일어나지 않는다. 그것을 유니온 마운트 시스템이 처리해 준다.

그럼 도커 데몬을 통해서 실제 컨테이너의 마운트 구조를 확인해 보자.

```shell
$ sudo docker inspect 14f80d300a86

"GraphDriver": {
  "Data": {
    "LowerDir": "/var/lib/docker/overlay2/3d8b7896e1f442c37d136c95c9e1bff87507ea075265e7da8ce48fdb285e8041-init/diff:/var/lib/docker/overlay2/a2a0ab8345486b3460f4da00b991e78841c8b0bd1309670ba216d000fbe52ed2/diff:/var/lib/docker/overlay2/f890923256b36b956c95243f2090c43297f71946d9b46beb0ef3dbb9ed5e28e8/diff:/var/lib/docker/overlay2/81e4e3f739fcfd0cb11841bdbeb13ceb22984766ca5023fe5cbc3b74752be65c/diff:/var/lib/docker/overlay2/d40bf4be9d80e4d3806da1e81e26fa9e6ff3293a9fe739a046eae1563a6d8ae1/diff:/var/lib/docker/overlay2/1883068af87a13a348c682d02c21b2ed719a5ca654fafba6687c67966655f3fa/diff:/var/lib/docker/overlay2/796eb70633c9cfc1f0c786b47d43304419b55cde09b47fe826eac5474203aec3/diff",
    "MergedDir": "/var/lib/docker/overlay2/3d8b7896e1f442c37d136c95c9e1bff87507ea075265e7da8ce48fdb285e8041/merged",
    "UpperDir": "/var/lib/docker/overlay2/3d8b7896e1f442c37d136c95c9e1bff87507ea075265e7da8ce48fdb285e8041/diff",
    "WorkDir": "/var/lib/docker/overlay2/3d8b7896e1f442c37d136c95c9e1bff87507ea075265e7da8ce48fdb285e8041/work"
            },
  "Name": "overlay2"
},
```

**LowerDir에 이미지 레이어들이 포함되고 UpperDir이 컨테이너 레이어가 된다**. 컨테이너를 여러 개 실행해도 이미지 레이어만 공유할 뿐 각각의 컨테이너 전용 쓰기 레이어가 만들어지기 때문에 영향을 주지 않는다.

컨테이터는 기본적으로 이미지 계층의 파일들을 기반으로 실행되고, **최상단의 컨테이너 전용 레이어는 처음에는 텅 빈 상태**이다. 컨테이너에서 일어나는 모든 변경사항들은 최상단 레이어에 기록된다. 컨테이너를 종료시키면 변경된 파일들이 날아가는데 컨테이너에서 일어나는 모든 변경사항은 container-layer-14f80d300a86 레이어에 저장되고, 컨테이너가 삭제되면 같이 날려버린다. 그러니까 컨테이너 레이어는 기본적으로 휘발적으로 사용된다.

### 컨테이너 변경사항 확인

docker diff는 컨테이너 레이어의 변경 사항을 보여주는 명령이다. nginx:latest 이미지를 bash 셀 컨테이너를 실행해 보자.

```shell
root> docker run -it nginx:latest bash
root@14f80d300a86:/#
```

다른 창을 오픈하고 이 컨테이너의 docker diff를 실행해 보자.

```shell
$ sudo docker diff 14f80d300a86
```

아무 것도 출력되지 않으면 정상이다. 왜냐하면 아무런 변경사항이 없기 때문이다. docker inspect에서 현재 마운트된 파일의의 시스템의 UpperDir를 확인한다.

```shell
  "UpperDir": "/var/lib/docker/overlay2/c09efdfa0e9f21da3e6c70a87606053c21700874dd225b8fc4b59c88d3de64fc/diff",
                "WorkDir": "/var/lib/docker/overlay2/c09efdfa0e9f21da3e6c70a87606053c21700874dd225b8fc4b59c88d3de64fc/work"
```

UpperDir로 이동한다.

```shell
$ cd /var/lib/docker/overlay2/c09efdfa0e9f21da3e6c70a87606053c21700874dd225b8fc4b59c88d3de64fc/diff
$ ls
```

이제 컨테이너의 bash 셸에서 파일을 하나 만들어 본다.

```shell
root@14f80d300a86:/# cd tmp
root@14f80d300a86:/tmp# touch I_AM_CONTAINER
root@14f80d300a86:/tmp#
```

다시 docker diff 명령으로 변경된 내용을 확인한다.

```shell
3de64fc/diff# docker diff 14f80d300a86
C /tmp
A /tmp/I_AM_CONTAINER
```

앞의 문자는 변경된 내용을 의미한다. C는 변경, A는 추가이다. 레이어 디렉터리에서도 변경사항을 확인해 본다.

```shell
$ tree
.
└── tmp
    └── I_AM_CONTAINER

1 directory, 1 file
```

정확히 컨테이너에서 변경된 내용만이 저장되어있는 것을 확인할 수 있다. 이제 파일을 하나 삭제할 것이다. 다음과 같이 명령을 실행한다.

```shell
rm /bin/tar
```

tar 파일은 이미지 레이어에 있는 파일 이름이다. 이미지 레이어에 있는 파일을 삭제했지만 이것은 컨테이너에 기록된다. diff를 사용해서 다시 변경 내용을 확인해본다.

```shell
$ docker diff 14f80d300a86
C /tmp
A /tmp/I_AM_CONTAINER
C /bin
D /bin/tar
```

이번에는 D라는 문자열이 보인다. D는 파일이 삭제되었다은 의미이다. docker diff가 보여주는 것은 컨테이너 레이어의 변경사항이기 때문에 삭제 또한 컨테이너에 기록한다. 실제 컨테이너 레이어 디렉터리는 어떻게 저장이 되었는지 확인해보자.

```shell
$ tree
.
├── bin
│   └── tar
└── tmp
    └── I_AM_CONTAINER
```

tree나 ls를 실행시켜보면 색이 다르게 출력된다. 이 파일은 일반 파일이 아니라 Character device 형식으로 저장되며, 이는 OverlayFS에서 파일 삭제를 나타내는 특별한 파일이다. 이 파일은 하위 레이어의 파일이 존재하지 않는 것처럼 가려버리는 역할을 한다.

### 변경사항을 이미지로 저장

docker diff로 확인 가능한 변경 사항을 이미지로 저장하는 방법을 살펴보자. 이 역할을 수행하는 명령어가 docker commit이다. commit 명령어는 컨테이너 아이디와 새로운 이미지 이름을 인자 값으로 받는다.

```shell
$ docker commit 14f80d300a86 nginx:no_tar
sha256:d1f7b469f65f0ec0d9ada3f2fac0c7e914c49554657f900788f47e606ee51e16
```

그리고 이 이미지로 컨테이너를 실행하면 tar를 사용할 수 없다.

```shell
$ docker run -it nginx:no_tar bash
$ tar
bash: tar: command not found
```

docker 커밋 명령은 14f80d300a86 컨테이너의 컨테이너용 레이어 container-layer-14f80d300a86를 최상단 레이어로 가지는 이미지를 새로 생성한다. 따라서 새로 만든 nginx:no_tar 이미지 레이어의 구조는 다음과 같다.

```shell
container-layer-14f80d300a86
layer 3 - nginx:latest
layer 2
layer 1
```

위에서 만든 이미지는 4개의 읽기 전용 레이어로 구성되어있다. 이 이미지를 기반으로 컨테이너를 실행 시키면 이 위에 컨테이너 전용 쓰기 레이어가 추가된다.

```shell
container-layer-234525252cd0 // 새로 추가된 레이어
container-layer-14f80d300a86
layer 3 - nginx:latest
layer 2
layer 1
```

여기까지가 도커 이미지가 만들어지는 기본적인 과정이다. 컨테이너를 만들고 새로운 레이어에 변경 사항을 더하고 그 내용을 커밋해서 읽기 전용 레이어로 기존 이미지에 올려서 새로운 이미지를 만든다.

## 이미지 만들기

### 도커 커밋으로 이미지 만들기

ubuntu:focal 이미지를 받는다.

```shell
$ docker pull ubuntu:focal
```

이 이미지 위해 git를 설치해 보자. 받은 이미지를 실행하고, 실행된 이미지 안에서 apt-get을 실행한다.

```shell
$ docker run ubuntu:focal /bin/sh -c 'apt-get update' 
```

그리고 이 레이어를 ubuntu-git-layer-1 이미지로 저장한다.

```shell
$ docker commit $(docker ps -alq) ubuntu:git-layer-1
```

$(docker ps -alq)는 가장 최근에 만들어진 컨테이너 ID로 치환된다.

두 번째로 ubuntu:git-layer-1 이미지에서 apt-get install -y git 명령어로 git을 설치한다. 그리고 이 레이어를 ubuntu:git 이미지로 저장한다.

```shell
$ docker run ubuntu:git-layer-1 /bin/sh -c 'apt-get install -y git'
$ docker commit $(docker ps -alq) ubuntu:git
```

이 이미지에 git가 잘 설치되었는지 확인해보자.

```shell
$ docker run -it ubuntu:git bash -c 'git --version'

git version 2.25.1
```

정상적으로 git 명령어가 동작하는 것을 확인할 수 있다. docker h istory로 이미지의 계층을 살펴보자.

```shell
$ docker history ubuntu:git
IMAGE          CREATED              CREATED BY                                      SIZE      COMMENT
298e0e99d5e4   About a minute ago   /bin/sh -c apt-get install -y git               102MB
bbbcb883d109   2 minutes ago        /bin/sh -c apt-get update                       29.2MB
9873176a8ff5   10 days ago          /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>      10 days ago          /bin/sh -c #(nop) ADD file:920cf788d1ba88f76…   72.7MB
```

9873176a8ff5는 ubuntu:focal 이미지의 레이어이다. 그 위로는 방금 생성한 레이어들이 쌓여있는 것을 확인할 수 있다. 이미지를 만들 때 사용한 컨테이너에서 실행한 명령어도 정확히 기록된 것을 확인할 수 있다.

### Dockerfile로 이미지 만들기

이번에는 Dockerfile로 같은 이미지를 만들어보자.

```shell
FROM ubuntu:focal
RUN apt-get update
RUN apt-get install -y git
```

위 내용을 Dockerfile로 저장하고 build 명령어로 ubuntu:git2라는 이름의 이미지를 만들어 보자.

```shell
$ docker build -t ubuntu:git2 .
```

출력되는 내용이 상당히 많다. 도커 빌드 과정에서 실행되는 명령어의 출력 결과는 생략하고 docker build 명령어 자체의 출력 결과만 남겨보겠다.

```shell
root> docker build -t ubuntu:git2 .
Sending build context to Docker daemon  26.11kB
Step 1/3 : FROM ubuntu:focal
 ---> 9873176a8ff5
Step 2/3 : RUN apt-get update
 ---> Running in 3059298c885d
Get:1 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]

Fetched 18.4 MB in 7s (2478 kB/s)
Reading package lists...
Removing intermediate container 3059298c885d
 ---> 410c5af75564
Step 3/3 : RUN apt-get install -y git
 ---> Running in 2b4bc526e1bc

Removing intermediate container 2b4bc526e1bc
 ---> a319f7f07c4f
Successfully built a319f7f07c4f
Successfully tagged ubuntu:git2
```

'Step 1/3 : FROM ubuntu:focal'은 FROM ubuntu:focal을 실행하는 단계이다. FROM은 새로 생성하는 이미지의 베이스 이미지를 지정하는 명령어이다. 다음 줄에 있는 9873176a8ff5 다이제스트 값은 ubuntu:focal의 이미지 아이디이다.

'Step 2/3 : RUN apt-get update'는 apt-get update를 실행하는 단계이다. 무엇인가를 실행하고ㅗ 그 다음에 정확히 같은 다이제스트 값을 지우는데 컨테이너를 지웠다고 나온다. Dockerfile의 RUN은 단순히 셸명령어를 실행하는 작업이 아니다. 바로 전 스텝의 이미지를 기반으로 컨테이너를 실행하고 RUN에 지정된 명령어를 실행한다. 그리고 이 컨테이너를 커밋해서 새로운 이미지로 저장한다. 마지막 줄의 다이제스트 값이 바로 중간 이미지의 ID 값이다.

RUN 명령어가 하나 더 있으니 똑 같은 과정을 한 번 더 거친다. 이번에는 git를 설치한다. 이번에\[ 만들어진 이미지 아이디는 a319f7f07c4f이다. 이 이미지에 ubuntu:git2라는 이름을 태그해준다.

그럼 git가 정상적으로 설치되었는지 확인해보자.

```shell
$ docker run -it ubuntu:git2 bash -c 'git --version'
git version 2.25.1
```

docker history로 이미지 레이어들을 확인해 보면 이미지 아이디가 다른 것을 제외하곤 docker commit으로 이미지를 만든 것과 동일하다. 실제 애플리케이션을 도커라이즈할 때는 훨씬 더 긴 Dockerfile을 작성하게 됩니디만, 이미지가 생성되는 기본적인 원리는 동일하다.
