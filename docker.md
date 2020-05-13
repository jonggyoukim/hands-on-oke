
# 애플리케이션 작성

Node.JS로 만들어진 사용자를 관리하는 애플리케이션을 작성해 보도록 한다. 이 애플리케이션은 MySQL에 새로운 유저와 테이블을 사용한다.

## 소스 가져오기

애플리케이션의 소스는 미리 만들어진 것을 가져와서 사용하고, MySQL은 미리 만들어진 도커 이미지를 사용한다.

1. git 에서 기존에 만들어진 애플리케이션 소스를 가져옵니다.

    ~~~
    git clone https://github.com/jonggyoukim/hands-on-oke-sample
    ~~~

    hands-on-oke-sample 디렉토리로 이동을 한다.
    ~~~
    cd hands-on-oke-sample
    ~~~


## 데이터베이스 시작  ( ***📌 미리 설정했다면 건너뛰기*** )
<!--
1. 인스턴스 포트 열기

    ~~~sh
    sudo firewall-cmd --add-port=3306/tcp --permanent
    sudo systemctl restart firewalld
    ~~~
-->
1. MySQL 컨테이너 실행하기

    ~~~
    docker run --name mydb -e MYSQL_ROOT_PASSWORD=mypassword -p 3306:3306 -d shiftyou/oke-mysql 
    ~~~


## 애플리케이션 시작
<!--
1. 인스턴스 포트 열기

    ~~~sh
    sudo firewall-cmd --add-port=8080/tcp --permanent
    sudo systemctl restart firewalld
    ~~~
-->

1. 환경설정하기

    해당 애플리케이션은 MySQL에 접속하기 위해서 환경변수가 필요한다.  
    미리 기동해 놓은 mysql에 대한 설정을 한다.
    ~~~sh
    export MYSQL_SERVICE_HOST=localhost
    ~~~

1. 애플리케이션 실행하기  

    먼저 관련 패키지를 설치한다.
    ~~~sh
    npm install
    ~~~

    화면이 하나라서 백그라운드로 실행한다.
    ~~~
    npm start &
    ~~~

    다음과 같이 나오면 실행한 것이다.
    ~~~
    > oke-sample-app@1.0.0 start /home/jonggyou_k/hands-on-oke-sample
    > node app.js

    host:localhost
    user:test
    password:Welcome1
    database:sample
    Server running on port: 8080
    Connected to database
    ~~~

1. curl로 체크한다.

    현재는 클라우드쉘을 사용하여 클라우드의 네트워크를 사용할 수 없어 브라우저로는 접근이 불가능하다. 그래서 curl로 확인해 보도록 한다.
    ~~~
    curl localhost:8080
    ~~~

    HTML 소스가 보일 것이며 성공한 것이다.  
    

1. 마치기

    백그라운드로 실행하고 있는 애플리케이션을 종료하고 확인한다.

    ~~~
    kill %1
    jobs
    ~~~


# 애플리케이션을 도커 이미지로 만들기

이번 단계는 위에서 만든 애플리케이션을 도커 이미지로 만드는 과정이다.
도커 이미지를 만들기 위해서 Dockerfile 이 필요하며 미리 생성되어 있다.

1. Dockerfile 살펴보기

    ~~~
    cat Dockerfile
    ~~~

    **hands-on-oke-sample** 디렉토리 내에 Dockerfile 이 있다.  
    이 파일은 도커 이미지를 만들기 위한 설정파일이다.

    ~~~docker
    # Node 버젼 8의 이미지를 기본으로 한다.
    FROM node:alpine

    # 애플리케이션이 위치할 디렉토리를 생성한다.
    WORKDIR /user/src/app

    # npm을 이용하여 필요한 패키지를 설치한다.
    COPY package*.json ./
    RUN npm install

    # 모든 애플리케이션 파일을 복사한다.
    COPY . .

    # 포트를 익스포즈 한다.
    EXPOSE 8080

    # 애플리케이션를 실행한다.
    CMD ["npm", "start"]
    ~~~

1. 도커 이미지 만들기
    ~~~sh
    docker build -t oke-sample .
    ~~~

    다음과 같이 도커 이미지가 만들어집니다.
    
    ~~~sh
    Sending build context to Docker daemon  6.607MB
    Step 1/7 : FROM node:8-alpine
    Trying to pull repository docker.io/library/node ... 
    8-alpine: Pulling from docker.io/library/node
    e6b0cf9c0882: Pull complete 
    93f9cf0467ca: Pull complete 
    a564402f98da: Pull complete 
    b68680f1d28f: Pull complete 
    Digest: sha256:38f7bf07ffd72ac612ec8c829cb20ad416518dbb679768d7733c93175453f4d4
    Status: Downloaded newer image for node:8-alpine
    ---> 2b8fcdc6230a
    Step 2/7 : WORKDIR /user/src/app
    ---> Running in 42fb559539a9
    Removing intermediate container 42fb559539a9
    ---> 854077d73d54
    Step 3/7 : COPY package*.json ./
    ---> 2d07df7eb007
    Step 4/7 : RUN npm install
    ---> Running in 66fe120698d3
    ...
    ~~~

1. 이미지 확인하기

    만들어진 도커 이미지를 확인하기 위하여 다음의 명령을 내립니다.

    ~~~sh
    docker images
    ~~~

    다음과 같이 oke-sample 이 만들어져 있음을 알 수 있다.
    ~~~sh
    REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
    oke-sample           latest              26b2fdd8129f        9 seconds ago       82.5MB
    node                 8-alpine            2b8fcdc6230a        4 months ago        73.5MB
    shiftyou/oke-mysql   latest              c34069dbe43e        6 months ago        437MB
    ~~~

# 애플리케이션을 컨테이너에서 수행하기

앞서 만든 애플리케이션 이미지를 사용하여 컨테이너를 실행해 보도록 한다.


1. 애플리케이션 실행하기

    최종적으로  다음과 같이 애플리케이션을 실행한다.
    ~~~
    docker run --name app  -e MYSQL_SERVICE_HOST=mydb -e -d -p 8080:8080 sample-app
    ~~~

    1. 옵션설정 : 필요한 환경변수 대입하기

        이 애플리케이션은 필요한 환경변수 `MYSQL_SERVICE_HOST`가 있다. 이 환경변수는 mysql 이 서비스 되고 있는 호스트를 나타낸니다. 설정하지 않으면 `localhost`로 대입되어 해당 애플리케이션 이미지에는 mysql이 존재하지 않아 애플리케이션이 동작하지 않습니다. 여기에서는 mysql 이 mydb 라는 이름의 컨테이너로 수행되기에 mydb라고 명명한다.

        도커에서 환경변수의 값은 `-e` 옵션을 사용하여 대입한다.
        ~~~
        -e MYSQL_SERVICE_HOST=mydb
        ~~~

        여러개의 환경변수는 -e 를 계속 반복해 줍니다.
        ~~~
        -e MYSQL_SERVICE_HOST=129.213.149.203 -e MYSQL_SERVICE_USER=test -e MYSQL_SERVICE_PASSWORD=Welcome1 -e MYSQL_SERVICE_DATABASE=sample 
        ~~~

    1. 옵션설정 : 포트 포워딩

        해당 애프리케이션은 8000 포트로 서비스 됩니다. 그래서 같은 포트로 포워딩 하기위해 다음과 같은 옵션을 사용한다. 앞의 8000 은 호스트의 포트이고, 뒤의 8000은 컨테이너의 포트이다.
        ~~~
        -p 8080:8080
        ~~~

1. 컨테이너 정보보기

    현재 운영중인 Container를 출력한다.

    ~~~
    $ docker ps

    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
    9fc6ec72293b        sample-app          "docker-entrypoint.s…"   39 seconds ago      Up 38 seconds       0.0.0.0:8080->8080/tcp              app
    ~~~

    **9fc6ec72293b** 이라는 Container ID를 사용하여 해당 Container가 가지는 IP를 확인 한다.
    ~~~
    $ sudo docker inspect -f  "{{ .NetworkSettings.IPAddress }}" 컨테이너아이디

    172.18.0.3
    ~~~


1. 테스트 하기

    웹브라우저로 `http://호스트IP:8080/`을 접속해 봅니다.  
    **호스트IP**는 해당 인스턴스의 공유IP 이다.
    
    이전에 애플리케이션으로 수행한 화면과 동일하지만, 표시되는 IP Address가 VM의 IP Address가 아닌 컨테이너의 IP Address를 나타내고 있다.


    ![](images/app2.png)
    
    표시되는 IP Address가 현재 Docker Container의 IP Address를 나타내고 있다.
    
    이로써 애플리케이션을 도커이미지로 만들고, 컨테이너로 수행완료하였습니다.  


1. 종료하기
    1. 도커로 실행중인 mysql을 종료하고 확인한다.
        ~~~
        docker stop mydb
        docker rm mydb
        docker ps -a
        ~~~

---
완료하셨습니다.

