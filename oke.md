# OKE에 애플리케이션 배포하기

![](images/step4.png)


# Docker Registry 에 push 하기

도커 이미지를 푸시할 대상인 레지스트리는 두가지로 설명합니다.

- Docker Hub (hub.docker.com)
- OCIR (Oracle Cloud Infrastructure Registry)

## Docker Hub 에 푸시하기

1. login 하기

    기본적으로 push 는 docker.io 에 이미지를 push 합니다.  
    자신의 도커 이미지를 push 하기 위해서 docker.io 에 로그인을 합니다.

    ~~~
    docker login
    ~~~

    다음과 같이 id/pw를 입력하는 화면이 나오면 입력합니다.

    ~~~
    Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
    Username: <사용자ID>
    Password: <사용자PW>
    WARNING! Your password will be stored unencrypted in /home/jonggyou_k/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credentials-store

    Login Succeeded
    ~~~

1. tag 생성

    이전 항목에서 우리는 oke-sample 이라는 docker image를 생성하였습니다.  
    Docker Hub 내의 자신의 ID에 이미지를 두기 위해서는 tag명이 `<사용자ID>/<이미지명>` 이어야 합니다.

    ~~~
    docker tag oke-sample <사용자ID>/<이미지명>
    ~~~

    아래와 같이 확인하면 새로운 tag명으로 생성된 docker image가 보입니다.

    ~~~
    $ docker tag oke-sample shiftyou/oke-sample

    $ docker images

    REPOSITORY            tag                 IMAGE ID            CREATED              SIZE
    oke-sample            latest              b3851a172fd8        About a minute ago   82.5MB
    shiftyou/oke-sample   latest              b3851a172fd8        About a minute ago   82.5MB
    node                  8-alpine            2b8fcdc6230a        4 months ago         73.5MB
    ~~~

1. push 하기

    ~~~
    docker push <사용자ID>/<이미지명>
    ~~~

    아래와 같이 docker.io 에 `<사용자ID>/<이미지명>` 으로 push 하는 로그들이 나오고 완료됩니다.

    ~~~
    $ docker push shiftyou/oke-sample

    The push refers to repository [docker.io/shiftyou/oke-sample]
    9ad596b6507c: Preparing 
    89015aa9a76a: Preparing 
    7d76b70f6e3c: Preparing 
    8e27730a26d6: Preparing 
    6b73202914ee: Preparing 
    9851b6482f16: Waiting 
    540b4f5b2c41: Waiting 
    ...
    ~~~

    >누구나 사용 가능한 이미지로 Kubernetes 에서도 secret 정보 필요없이 사용 가능합니다.

## OCIR 에 푸시하기

1. 대시보드

    좌측 상위메뉴에서 `개발자서비스 > 레지스트리(OCIR)`을 선택하면 다음과 같은 화면이 보여집니다.

    ![](images/oke3.png)

1. token 만들기

    Private한 환경인 OCIR에 이미지를 push 하기 위해서는 인증토큰이 필요합니다.

    우측 상단의 ![프로파일](images/profile-icon.png) 아이콘을 클릭하고 `사용자 설정` 을 선택합니다.

    ![](images/oke1.png)

    좌측의 리스트에서 `인증 토큰`을 선택합니다.

    ![](images/oke2.png)

    `토큰 생성`을 선택하여 설명부분에 설명을 쓰고 토큰을 생성합니다.  
    한번 생성된 토큰은 다시 볼 수 없기에 생성되고 나면 복사를 해 둡니다.

1. login 하기

    OCIR에 로그인은 다음과 같이 합니다.

    ~~~
    docker login <리젼키>.ocir.io
    ~~~

    현재 Seoul 리젼을 쓰기 때문에 리젼키는 `icn` 이 됩니다.
    > 다른 리젼을 사용하면 [지역별 가용성](https://docs.cloud.oracle.com/en-us/iaas/Content/Registry/Concepts/registryprerequisites.htm#Availab)을 참조바랍니다.

    ~~~
    docker login icn.ocir.io
    ~~~

    입력할 항목은 영역에는 다음을 입력합니다.
    ~~~
     Username : <태넌시-네임스페이스>/<사용자ID>
     Password : <토큰>
    ~~~

    입력사항은 다음에서 찾을 수 있습니다.

    - \<태넌시-네임스페이스\>
    
      좌측 상위메뉴에서 `개발자서비스 > 레지스트리(OCIR)`을 선택하면 다음과 같은 화면이 보여집니다.

      ![](images/oke4.png)

      위 화면에서 빨간 박스 부분인 `cnriar6c4yj1` 라는 명이 **\<태넌시-네임스페이스\>** 입니다. 이를 이용하여 로그인을 합니다.
    
    - \<사용자ID\>

      우측 상단의 ![프로파일](images/profile-icon.png) 아이콘을 클릭하고 `사용자 설정` 을 선택합니다.

      ![](images/oke1.png)

      ![](images/oke5.png)
      
      위의 화면에서 빨간 박스 부분인 `oracleidentitycloudservice/jonggyou.kim@oracle.com` 라는 명이 **\<사용자ID\>** 입니다.  

      
      간단히 `jonggyou.kim@oracle.com` 으로 나타나지 않고 `oracleidentitycloudservice/jonggyou.kim@oracle.com` 로 나타나는 이유는 Oracle Identity Cloud Service 에서 연합된 계정의 경우 `oracleidentitycloudservice/` 가 추가 되었기 때문입니다.

    Username 항목에는 위의 <태넌시-네임스페이스>/<사용자ID> 를 입력합니다. 위의 화면을 예를 들면 다음과 같습니다.

    ~~~
    cnriar6c4yj1/oracleidentitycloudservice/jonggyou.kim@oracle.com
    ~~~

    그리고 패스워드는 이전에 기록한 토큰을 사용합니다.

    ~~~
    $ docker login icn.ocir.io

    Username: cnriar6c4yj1/oracleidentitycloudservice/jonggyou.kim@oracle.com
    Password: 
    WARNING! Your password will be stored unencrypted in /home/jonggyou_k/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credentials-store

    Login Succeeded
    ~~~

1. tag 생성

    OCIR에 이미지를 두기 위해서는 이미지의 태그가 `<리젼키>.ocir.io/<태넌시-네임스페이스>/<이미지명>:<태그>` 이어야 합니다.  
    사용자별 또는 프로젝트별로 구분하기 위하여 `<이미지명>`을 `<사용자명>-<이미지명>` 으로 구성하도록 합니다.

    ~~~
    docker tag oke-sample <리젼키>.ocir.io/<태넌시-네임스페이스>/<이미지명>:<태그>
    ~~~
    - <리젼키> : icn
    - <태넌시-네임스페이스> : cnriar6c4yj1
    - <이미지명> : jonggyou-oke-sample

    아래와 같이 tag를 주도록 합니다.

    ~~~
    $ docker tag oke-sample icn.ocir.io/cnriar6c4yj1/jonggyou-oke-sample
    ~~~

    확인하면 새로운 tag명으로 생성된 docker image가 보입니다.

    ~~~
    $ docker images

    REPOSITORY                                     tag                 IMAGE ID            CREATED             SIZE
    shiftyou/oke-sample                            latest              b3851a172fd8        2 hours ago         82.5MB
    icn.ocir.io/cnriar6c4yj1/jonggyou-oke-sample   latest              b3851a172fd8        2 hours ago         82.5MB
    oke-sample                                     latest              b3851a172fd8        2 hours ago         82.5MB
    node                                           8-alpine            2b8fcdc6230a        4 months ago        73.5MB
    ~~~



1. push 하기

    해당 이미지를 OCIR에 push 합니다.

    ~~~
    $ docker push icn.ocir.io/cnriar6c4yj1/jonggyou-oke-sample

    The push refers to repository [icn.ocir.io/cnriar6c4yj1/jonggyou-oke-sample]
    9ad596b6507c: Pushed 
    89015aa9a76a: Pushed 
    7d76b70f6e3c: Pushed 
    8e27730a26d6: Pushed 
    6b73202914ee: Pushed 
    9851b6482f16: Pushed 
    540b4f5b2c41: Pushed 
    6b27de954cca: Pushed 
    latest: digest: sha256:bdc0574ee9b802e075df623f052a3d7b981fcb4e883d6d67e8489de445407521 size: 1993
    ~~~

    다음과 같이 OCIR에 등록되었음을 알 수 있습니다.

    ![](images/oke6.png)

    >OCIR은 인증된 사용자만 pull 할 수 있는 이미지로 Kubernetes에서는 sercret 정보 설정이 필요합니다.


# OKE에 배포하기

OKE는 오라클에서 제공하는 쿠버네티스 환경입니다. 따라서 여타 다른 쿠버네티스 운영과 동일합니다.

현재 `yaml` 디렉토리에 보면 배포를 위한 yaml 파일이 두개 있습니다.

- oke-mysql.yaml : DB 배포를 위한 yaml
- oke-sample.yaml : APP 배포를 위한 yaml

위 두개의 yaml 파일로 배포를 합니다.

## MySQL 배포하기

MySQL은 핸즈온에서 공통적으로 사용할 예정이라 1개를 미리 실행해 놓았습니다. 

<details>
<summary> 📌 실행되어 있지 않다면 여기를 클릭하여 실행합니다.</summary>
<div markdown="1">

  oke-mysql.yaml은 다음의 내용을 포함합니다.

  ~~~yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: oke-mysql
    namespace: default
    labels:
      app: oke-mysql
  spec:
    selector:
      matchLabels:
        app: oke-mysql
    strategy:
      type: Recreate
    template:
      metadata:
        labels:
          app: oke-mysql
      spec:
        containers:
          - name: oke-mysql
            image: shiftyou/oke-mysql
            ports:
            - containerPort: 3306
              name: oke-mysql
            env:
            - name: MYSQL_ROOT_PASSWORD
              value: mypassword
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: oke-mysql
    namespace: default
    labels:
      app: oke-mysql
  spec:
    ports:
    - port: 3306
    selector:
      app: oke-mysql
    type: ClusterIP
  ~~~

  1. Deployment 

      - 애플리케이션의 이름은 oke-mysql 으로 합니다.
      - 이미지는 shiftyou/oke-mysql 이미지를 사용합니다.
      - 포트는 3306번을 사용합니다.
      - 환경변수로 MYSQL_ROOT_PASSWORD 값으로 mypassword를 사용합니다.
      - namespace는 default로 합니다.


  1. Service

      - oke-mysql 로 명명된 Deployment를 Service 로 노출합니다.
      - 서비스는 ClusterIP 타입입니다.
      - 포트는 3306번입니다.
      - namespace는 default로 합니다.


  1. 배포

      MySQL은 각자 배포하지 않고 하나만 배포하도록 합니다. 그래서 default 네임스페이스에 배포하도록 구성되어 있습니다.

      만약 두번째 이상 배포하려 한다면 이미 배포되었기 때문에 변경사항이 없습니다.

      배포는 다음과 같이 합니다.

      ~~~
      kubectl apply -f oke-mysql.yaml
      ~~~

      다음과 같이 현재 배포 상태를 볼 수 있습니다.

      ~~~
      kubectl get all -n default
      ~~~

      `-n default`는 현재 oke-mysql 이름의 서비스가 default 라는 이름의 네임스페이스에 배포가 되도록 구성되고 배포되어 있습니다. 그래서 default 라는 이름의 네임스페이스의 상태를 보기 위해서 `-n default`를 옵션으로 줍니다.

      옵션을 주지 않으면 이전에 `kubectl config set-context --current --namespace jonggyou` 라고 명령하여 변경된 기본 네임스페이스를 사용합니다.

      결과로 다음과 같이 출력됩니다.
      
      ~~~
      NAME                             READY   STATUS    RESTARTS   AGE
      pod/oke-mysql-6d4675d7f6-v5fkh   1/1     Running   0          2m18s

      NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
      service/kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP    3d6h
      service/oke-mysql    ClusterIP   10.96.49.43   <none>        3306/TCP   2m18s

      NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
      deployment.apps/oke-mysql   1/1     1            1           2m18s
    
      NAME                                   DESIRED   CURRENT   READY   AGE
      replicaset.apps/oke-mysql-6d4675d7f6   1         1         1       2m18s
      ~~~

</div>
</details>



## 애플리케이션 배포하기

oke-sample.yaml 은 다음의 내용을 포함합니다.

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oke-sample
  labels:
    app: oke-sample
spec:
  selector:
    matchLabels:
      app: oke-sample
  template:
    metadata:
      labels:
        app: oke-sample
    spec:
      containers:
      - name: oke-sample
        image: shiftyou/oke-sample
        env:
        - name: MYSQL_SERVICE_HOST
          value: oke-mysql.default
        ports:
        - containerPort: 8080
          name: oke-sample
---
apiVersion: v1
kind: Service
metadata:
  name: oke-sample
  labels:
    app: oke-sample
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: oke-sample
  type: LoadBalancer
~~~


1. Deployment 

    - 애플리케이션의 이름은 oke-sample 으로 합니다.
    - 이미지는 shiftyou/oke-sample 이미지를 사용합니다.
    - 포트는 8008번으을 사용합니다.
    - 환경변수로 MYSQL_SERVICE_HOST 값으로 oke-mysql.default를 사용합니다.
    - namespace는 현재 기본 namespace를 합니다.


1. Service

    - oke-sample 로 명명된 Deployment를 Service 로 노출합니다.
    - 서비스는 LoadBalancer 타입입니다.
    - 포트는 80번을 사용합니다. 타겟포트는 8080입니다.
    - namespace는 현재 기본 namespace를 합니다.


1. 배포

    yaml 파일에서 보듯이 mysql의 주소를 `oke-mysql.default` 라고 하였습니다. 이는 default 네임스페이스에 서비스 하고 있는 oke-mysql 를 지정하는 것입니다.

    다음과 같이 배포합니다.

    ~~~
    kubectl apply -f oke-sample.yaml
    ~~~

    그리고 상태는 oke-mysql을 배포했을 때와 다르게 `-n default`라는 옵션이 필요없다. 굳이 한다면 `-n jonggyou`로 하지만 안 해도 기본으로 지정한 jonggyou 네임스페이스를 사용합니다.

    ~~~
    kubectl get all
    ~~~

    다음과 같이 잘 배포되어 서비스됨이 출력됩니다.
    
    ~~~
    NAME                              READY   STATUS    RESTARTS   AGE
    pod/oke-sample-5d59bb9596-wgk6n   1/1     Running   0          11s

    NAME                 TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
    service/oke-sample   LoadBalancer   10.96.6.202   <pending>     80:30151/TCP   11s

    NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/oke-sample   1/1     1            1           11s
    
    NAME                                    DESIRED   CURRENT   READY   AGE
    replicaset.apps/oke-sample-5d59bb9596   1         1         1       11s
    ~~~
    
    출력중에 Service 부분을 보면 LoadBalancer 를 사용하는데, 아직 EXTERNAL-IP는 \<pending>으로 나타납니다.

    ~~~
    NAME                 TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
    service/oke-sample   LoadBalancer   10.96.6.202   <pending>     80:30151/TCP   11s
    ~~~
    
    이는 OCI의 로드밸런서 서비스를 프로비져닝 하고 있음을 의미합니다.  
    다시 service 에 대해서 설펴보도록 합니다.

    ~~~
    kubectl get svc
    ~~~
    
    그러면 다음과 같이 EXTERNAL-IP가 나타납니다.
    
    ~~~
    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
    oke-sample   LoadBalancer   10.96.6.202   140.238.27.54   80:30151/TCP   5m41s
    ~~~

# 테스트 하기

앞서 본 oke-sample 애플리케이션은 로드밸런서로 서비스 중이며, 이 아이피는 EXTERNAL-IP인  140.238.27.54 으로 나타납니다. 그리고 포트는 80 포트로 서비스 중이며 내부에서는 8080으로 포워딩 합니다.

브라우저를 사용하여 140.238.27.54:80 로 접속해 보도록 합니다.

![](images/oke7.png)

몇 개의 연락처를 등록하면 다음과 같이 화면이 됩니다.

![](images/app.png)

---
완료하셨습니다. <a href="javascript:history.back();">뒤로가기</a>
