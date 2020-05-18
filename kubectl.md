# 애플리케이션 관리하기


## 애플리케이션 상태보기
- 로그

- pod 삭제

## 애플리케이션 확장하기

- 확장

- pod 삭제

- 축소


## 애플리케이션 업그레이드



## 데시보드
클러스터를 생성할 때 데시보드를 구성하게끔 하였다.  
데시보드는 기본적으로 ClusterIP로 배포되어 외부에서 접근이 불가하다.  
이를 변경하여 외부에서 접근가능하도록 한다.

1. 데시보드를 LoadBalancer 서버스로 변경한다.

    ~~~
    $ kubectl edit svc kubernetes-dashboard -n kube-system
    ~~~
    type 을 `LoadBalancer` 로 변경한다.

    외부 IP를 확인한다.

    ~~~
    $ kubectl get svc -n kube-system
    ~~~



1. 브라우저로 접속한다. 
 
    접속하면, 로그인을 하기 위하여 토큰을 입력해야 한다.
    
    ![](images/dashboard1.png)

1. 토큰을 얻는다.

    토큰은 다음과 같이 얻을 수 있다. 

    ~~~
    $ TOKENNAME=`kubectl -n kube-system get serviceaccount/kubernetes-dashboard -o jsonpath='{.secrets[0].name}'`

    $ echo $TOKENNAME

    $ TOKEN=`kubectl -n kube-system get secret $TOKENNAME -o jsonpath='{.data.token}'| base64 --decode`

    $ echo $TOKEN
    ~~~

    출력된 토튼의 값을 복사하여 로그인을 한다.

    로그인이 완료되면 다음과 같이 화면이 나온다. 
    
    ![](images/dashboard2.png)

>    만약 권한이 없다는 오류가 뜨면 다음과 같이 명령한다.
>
>    ~~~
>    $ kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
>    ~~~



---
완료하셨습니다. <a href="javascript:history.back();">뒤로가기</a>
