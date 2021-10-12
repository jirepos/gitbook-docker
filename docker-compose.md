# Docker Compose

## 설치

```shell
sudo apt install docker-compose
```

## 버전 확인

```shell
docker-compose -v
```

1.25 버전이 출력된다.

```shell
docker-compose version 1.25.0, build unknown
```

## 최신버전 설치

최신 버전 확인은 [Github](https://github.com/docker/compose/releases)에서 확인한다.

최신버전은 다음 명령으로 다운로드 받는다.

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

실행권한을 부여한다.

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

/usr/bin에 심볼릭 링크 걸기

```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

설치확인

```shell
$ docker-compose --version
docker-compose version 1.29.2, build 1110ad01
```

자세한 설명은 [공식사이트](https://docs.docker.com/compose/install/)를 참조한다.

## 서비스 정의 YAML 파일 작성

### 버전 정의

먼저 스키마 버전을 정의한다. 최신 버전을 사용하는 것이 좋다.

```yaml
version: "3.7" 
```

### App 서비스 정의

도커 컴포넌트로 생성할 서비스(도는 컨테이너) 옵션을 정의한다.

```yaml
version: "3.7" 
services:
```

먼저 서비스 항목과 컨테이너 이미지를 정의한다. 서비스 이름은 임의로 선택할 수 있다. 이름은 자동으로 네트워크 별칭이되므로 MySQL서비스를 정의할 때 유용하다.

```yaml
version: "3.7" 

services:
  app:
    image: node:12-alpine
```

순서 지정에 대한 요구 사항은 없지만 일반적으로 image 정의와 가까운 곳에 명령이 표시된다. 해당 명령을 파일로 이동한다.

```yaml
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
```

서비스의 ports를 정의하여 명령의 -p 3000:3000 부분을 마이그레이션한다.여기서는 짧은 구문을 사용하지만, 더 자세한 긴 구문을 사용할 수도 있다.

```yaml
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
```

다음에는 working_dir 및 volumes 정의를 사용하여 작업 디렉터리(-w /app)와 볼륨 매핑(-v ${PWD}:/app)을 둘 다 마이그레이션한다. 볼륨에도 짧은 구문과 긴 구문이 있다.

Docker Compose 볼륨 정의의 이점 중 하나는 현재 디렉터리의 상대 경로를 사용할 수 있다는 것이다.

```
services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
```

마지막으로, environment 키를 사용하여 환경 변수 정의를 마이그레이션한다.

```yaml
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
```

### MySQL 서비스 정의

이제 MySQL 서비스를 정의할 차례입니다. 해당 컨테이너에 사용한 명령은 다음과 같습니다(Windows PowerShell에서 \ 문자를 \`로 바꿈).

```shell
docker run -d \
 --network todo-app --network-alias mysql \
 -v todo-mysql-data:/var/lib/mysql \
 -e MYSQL_ROOT_PASSWORD=secret \
 -e MYSQL_DATABASE=todos \
 mysql:5.7
```

먼저 새 서비스를 정의하고 이름을 mysql로 지정하여 네트워크 별칭을 자동으로 가져옵니다. 사용할 이미지도 지정합니다.

```yaml
version: "3.7"

services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
```

다음에는 볼륨 매핑을 정의한다. docker run을 사용하여 컨테이너를 실행한 경우 명명된 볼륨이 자동으로 생성되었다. 그러나 Compose로 실행하는 경우에는 자동으로 생성되지 않는다. 최상위 volumes: 섹션에 볼륨을 정의한 다음, 서비스 구성에서 탑재 지점을 지정해야 합니다. 단순히 볼륨 이름만 지정하면 기본 옵션이 사용된다. 하지만 훨씬 더 많은 옵션을 사용할 수 있다.

```yaml
version: "3.7"

services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql

volumes:
  todo-mysql-data:
```

마지막으로, 환경 변수만 지정하면 된다.

```yaml
version: "3.7"

services:
  app:
    # The app service definition
  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

이 시점에서 전체 docker-compose.yml은 다음과 같이 표시되어야 한다.

```yaml
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

## 애플리케이션 스택 실행

이제 docker-compose.yml 파일이 준비되었으므로 시작할 수 있다.

docker-compose up 명령을 사용하여 애플리케이션 스택을 시작한다. 모든 명령을 백그라운드에서 실행하려면 -d 플래그를 추가한다. 또는 Compose 파일을 마우스 오른쪽 단추로 클릭하고 VS Code 사이드바에 대해 Compose 시작 옵션을 선택할 수 있다.

```shell
docker-compose up -d
```

## Volumes

현재 디렉터리 아래에 docker/data 디렉터리를 /var/lib/postgresql/data로 볼륨을 구성했다.

```yaml
services:
  db:
    image: postgres
    volumes:
      - ./docker/data:/var/lib/postgresql/data
```

이 디렉터리를 직접 관리하지 말고 도커에 맡길 수도 있다. docker-compose.yml 파일을 다음과 같이 수정한다.

```yaml
# 변경 부분!
volumes:
  django_sample_db_dev: {}

services:
  db:
    image: postgres
    volumes:
      # 여기도!
      - django_sample_db_dev:/var/lib/postgresql/data
```

처음 바뀐 부분에서는 django_sample_db_dev라는 이름으로 볼륨을 하나 만듭니다. 이렇게 만들어진 볼륨은 docker volume ls 명령으로 확인할 수 있습니다. (볼륨은 도커가 관리하는 가상 디스크라고 생각하면 됩니다.)

이렇게 만든 볼륨을 db 서비스에서 사용하려면, db 서비스 선언부 안에 volumes 항목을 넣고 - 가상디스크\_이름:컨테이너\_속\_디렉터리처럼 지정합니다. 이후로는 모든 데이터베이스 데이터가 django_sample_db_dev 볼륨에 저장됩니다. (이 볼륨을 지우려면 docker volume rm django_sample_db_dev 명령을 사용합니다.)
