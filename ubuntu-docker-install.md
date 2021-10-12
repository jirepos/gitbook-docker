# Ubuntu 환경

Docker는 Linux 환경에서 실행되기 때문에 Linux에서는 명령어만으로도 간단히 설치할 수 있다.

## Docker 설치

먼저 다음 명령을 단계적으로 실행한다.

```shell
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-cache policy docker-ce
```

## Docker 설치

Docker를 설치한다.

```shell
sudo apt install docker-ce
```

## Docker가 실행 중인지 확인

다음 명령을 실행하여 Docker가 실행 중인지 확인한다.

```shell
sudo systemctl status docker
```

Windows에서 WSL로 우분투를 설치한 경우에는 systemctl이 없다. 다음과 같이 service 명령을 사용한다.

```shell
service docker status
```

pull할 때 다음과 같은 오류가 난다.

```shell
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

그러면 docker 를 시작한다.

```shell
service docker start
```
