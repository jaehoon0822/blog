---
title: 'docker compose 에 대해서'
date: '2023-06-12'
tags: ['docker', 'docker-compose']
draft: false
summary: 'docker compose 에 대해서'
---

# `Docker compose` 에 대해서 알아본다.

`Docker` 를 컨테이너화 하다보면, 여러개의 `Container` 를 다루어야 할 일이 생긴다.

예를들면 `backend`, `frontend`, `mariadb` 같은 경우 말이다.
그때마다, `DockerFile` 을 각 `Diractory` 마다 지정하고, 각자  
`Docker build` 하는건 조금 아닌듯 하다.

역시나 이러한 불편함을 없애기 위해 여러 `Container` 를 한번에 다룰수 있는 `Docker compose` 가 존재한다.

`Docker compose` 는 여러개의 최상의 문이 존재한다.

- version:  
Docker-compose 의 버젼이다.  
버전에 따라, 형식의 변화가 있을 수 있으니 명시하는것이 좋다고 한다.

- services:  
Dokcer Container 를 `Build` 하기 위해 명시해주는 부분이다.  
`Docker-compose` 는 컨테이너 생성구문을 `service` 개념의 단위로 작성한다.  

- volumes:  
`Docker` 의 `volumn` 들을 정의한다.  

- configs:  
`Docker` 의 `Service` 에서 사용될 `ConfigFile` 을 정의한다.

- secrets:  
노출되서는 안되는 민감한 데이터에 대한 구성 데이터를 정의한다.  
`SSH`, `TLS Auth key`, `DB Auth data` 등등.. 컨테이너가 중지되면  
함께 제거된다.  
만약, `secrets` 정보가 필요하다면, `Docker secret create` 를 통해 생성가능하다.

- networks:  
`Docker container` 생성시 `network` 설정이 가능한데, 이러한 `network`  
를 명시하여 작성하는 부분이다.

다음은, `Docker Docs` 에서 제공하는 `file` 내용이다.

```yml

services:
  frontend:
    image: awesome/webapp
    ports:
      - "443:8043"
    networks:
      - front-tier
      - back-tier
    configs:
      - httpd-config
    secrets:
      - server-certificate

  backend:
    image: awesome/database
    volumes:
      - db-data:/etc/data
    networks:
      - back-tier

volumes:
  db-data:
    driver: flocker
    driver_opts:
      size: "10GiB"

configs:
  httpd-config:
    external: true

secrets:
  server-certificate:
    external: true

networks:
  # The presence of these objects is sufficient to define them
  front-tier: {}
  back-tier: {}
```

`docker compose up` 의 명령을 실행하면, 먼저 현재 `WorkDir` 에서 `docker-compose-yml` 파일을 찾는다.

```sh
docker compose up;
```

이렇게 실행하면 `--attach` 모드로 시작되므로, `-d` 를 붙이면,  
`--detach` 모드로 실행된다.

```sh
docker compose up -d;
```

만약 중지하고 싶다면, `stop` 옵션을 사용한다.

```sh
docker compose stop;
```

`docker compose` 로 실행한 컨테이너를 중지하고 삭제까지 하고 싶다면, `down` 옵션을 사용한다.

```sh
docker compose down;
```

다시 재시작 하고 싶다면, `start` 를 사용한다.

```sh
docker compose start;
```

또한 실행중인 `Compose` `list` 를 볼고 싶다면 `ls` 를 사용하면 된다.

```sh
docker compose ls;
```

컨테이너를 보고 싶다면 `ps` 를 사용한다.

```sh
docker compose ps;
```

이 외에도 정말 많은 `commend options` 들이 많다.
이를 살펴보려면 [docker compose CLI](https://docs.docker.com/compose/reference/) 를 보도록 하자.

`Docker` 는 기본값으로 `bridge` 라는 `drive` 로 가상네트워크 환경을 만들어 서로가 통신할 수 있도록 만든다.  

이러한 가상네트워크는 `docker network create [OPTIONS] NETWORK` 을 통해 생성이 가능하며, `-d (--dirive)` 를 통해 원하는 `network drive` 선택도 가능하다.

> `network dirve` 관련해서는 아직 지식이 많이 부족하여 각 방식의 차이는 이해하지 못하고 있다.

여러개의 분산된 `container` 는 `Docker engine` 으로 부터 `virtual IP` 를 할당받고, 같은 `Docker network` 를 통해 서로 통신하게 된다.

또한 `Container` 가 교체되어 `IP` 주소가 변경된다 하더라도 `Docker` 자체에서 `DNS`를 이용해 서비스 디스커버리 기능을 제공하여 문제 없이 작동된다.

이를 확인하기 위해 다음의 `compose file` 이 존재한다고 가정해보자

```yml
version: "3"
services:
  mariadbversion: "3"
services:
  mariadb:
    image: "mariadb"
    restart: always
    volumes:
      - data:/var/lib/mysql
    env_file:
      - ./env/mariadb.env
    ports:
      - 3306
  backend:
    build:
      context: .
      dockerfile: ./dockerfiles/backend.Dockerfile
    container_name: backend
    ports:
      - 8080:8080
    volumes:
      - ./backend:/app
  frontend:
    build:
      context: .
      dockerfile: ./dockerfiles/front.Dockerfile
    container_name: frontend
    ports:
      - 3000:3000
    volumes:
      - ./frontend:/app
volumes:
  data:
:
    image: "mariadb"
    # container_name: db
    restart: always
    volumes:
      - data:/var/lib/mysql
    env_file:
      - ./env/mariadb.env
    ports:
      - 3306
  backend:
    build:
      context: .
      dockerfile: ./dockerfiles/backend.Dockerfile
    container_name: backend
    ports:
      - 8080:8080
    volumes:
      - ./backend:/app
  frontend:
    build:
      context: .
      dockerfile: ./dockerfiles/front.Dockerfile
    container_name: frontend
    ports:
      - 3000:3000
    volumes:
      - ./frontend:/app
volumes:
  data:

```

그리고 `docker compose up -d` 를 실행시켜, 각 `container` 를 실행시켰다.  
이렇게 실행시킨다음, `docker -it frontend sh` 로 `frontend` 에 들어간후
`nslookup` 을 실행시킨다.

이에 대한 결과는 다음과 같다.

```sh
> nslookup frontend

Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:   frontend
Address: 172.19.0.3


> nslookup backend
Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:   backend
Address: 172.19.0.2

> nslookup mariadb
Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:   mariadb
Address: 172.19.0.4
```

각 할당된 `AddressIP` 와 함께, `DNS` 이름으로 `service` 이름이 할당되어서 접근되는것을 볼 수 있다.

만약 `db` 구성을 `2` 개로 만들어보면 다음처럼 작동된다.

```sh
> docker compose up -d --scale mariadb=2
```

```sh
> nslookup mariadb
Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:   mariadb
Address: 172.19.0.5
Name:   mariadb
Address: 172.19.0.4
```

이건 시험용으로, 만든것으로 실상 `mariadb` 에는 포트번호를 붙혀줘서  
`backend` 에서 접근해야 한다.

이건 예시일뿐이므로, 단순히 `scale` 을 통해 여러개의 `mariadb` 를 생성한것 뿐이다.

이때 제공되는 것을 보면 2개의 `db` 가 만들어진것을 볼 수 있다.
`AddressIP` 별로 다르다.  

하지만, `DNS` 조회시, 이 2개의 컨테이너가 전부 포함되는것을 볼 수 있다.
이것을 이용하면, 컨테이너에 트래픽을 고르게 분산되도록 만드는 로드벨런싱을 구현할 수 있다.

굉장히 흥미롭다.

실상 사용하는것 자체는 어렵지는 않다.
해당 내용에 맞추어 짜 들어가면 된다.

`volumes` 부분은, `service` 에 들어간 `volume list` 가 `named volume` 이라면 `volumes` 부분에 명시해주어야 한다.

만약, 이미 `docker volume create VOLUMENAME` 을 사용해 해당하는 `volume` 이 이미 존재한다면, `external flag` 를 사용하여, 명시하면 해당 `volume` 을 사용하는것으로 알고 있다.

이는 `networks` 부분도 마찬가지이다.

사용하는 부분에서 약간 헷갈린 부분이 `secrets` 부분이다.
`Docs` 의 예시를 보면 `secrets` 는 `server-certificate` 로 명시되어 있고, `external` 은 `ture` 이다.

이는 `secrets` 에서 사용되는 `server-certificate` 가 이미 생성되어 있으므로, 생성된 `secrets` 를 사용하겠다는 말이다.

사실, `.env` 파일에서 값을 가져와서 `configFile` 로 설정하면 되는거 아닌가? 싶은 마음이 들기도 하지만, `.env` 보다 더 안전하게 보호할수 있는것같다.

이유는 값 자체를 아예 암호화시켜 버린다.
다음을 보자.

```sh
> printf "my super secret password" | docker secret create my_secret -
onakdyv307se2tl7nl20anokv
```

이렇게 하면 `printf` 의 내용을 암호화해서 `my_secret` 에 들어간다.

```sh
> docker secret ls

ID                          NAME                CREATED             UPDATED
onakdyv307se2tl7nl20anokv   my_secret           6 seconds ago       6 seconds ago
```

리스트를 확인해보면 제대로 생성된것을 볼 수 있다.
이뿐만 아니라 `file` 을 저장할 수도 있다.

```sh
> docker secret create my_secret ./secret.json
dg426haahpi5ezmkkj5kyl3sn

> docker secret ls

ID                          NAME                CREATED             UPDATED
dg426haahpi5ezmkkj5kyl3sn   my_secret           7 seconds ago       7 seconds ago

```

이렇게 생성된, `secret` 인 `server-certificate` 사용한다는 것이다.
사용되는 부분은 `Docs` 의 예시중 `frontend` 쪽에 나와 있는것을 확인할 수 있다.

이부분을 제대로 활용하는 방법은 [Docker Swarm에서 시크릿(Secret)으로 패스워드 등 보안 정보 다루기](https://seongjin.me/docker-swarm-secret/) 에서 더 살펴볼 필요가 있을 듯 싶다.

여기서 `MYSQL` 을 사용해 `Secret` 을 사용한 예시가 나오는데, 유용해 보인다.