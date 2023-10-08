## configmap 사용
- 쿠버네티스 안에서 실행되는 애플리케이션에 설정 데이터를 전달하는 방법중 하나가 configmap를 사용하는 것이다.
- 쿠버네티스에서는 설정 옵션을 컨피그맵이라 부르는 별도 오브젝트로 분리할 수 있다. 
- configmap은 키/값 쌍으로 구성으로 이루어져있다.
- configmap에 정의한 값을 프로세스의 명령줄 인자로 전달할 수 있다. 


## 사용시 주의할 점
- configmap의 키값은 유효한 DNS 서브도메인이어야 한다.
- 영숫자, 대시, 밑줄, 점만 포함할 수 있다.


## command configmap 만들기
```agsl
kubectl create configmap testconfigmap --from-literal=america=english

여러 키값 넣어주기 
kubectl create configmap testconfigmap --from-literal=america=english --from-literal=canada=english 

파일 내용으로 configmap 만들기

kubectl create configmap fileconfig --from-file=config_file.conf

파일을 특정 키값에 넣어주기 
kubectl create configmap fileconfig --from-file=nginx.conf=config_file.conf
```


## pod에서 configmap가져오기 
```agsl
apiVersion: v1
kind: pod
metadata:
  name: test-pod
spec:
  containers:
  - image: test/test
  env:
  - name: NUMBER ---> NUMBER라는 환경변수에 설정
    valueFrom:
      configMapKeyRef:
        name: test-config  ----> 참조하는 configmap name 
        key: NUMBER  -----> configmap에 설정된 키 값을 number에 설정 
```


## 존재하지 않는 configmap을 파드내에 설정하면 파드가 죽을까? 
- 맞다 존재하지않는 컨피그맵을 파드가 참조한다면 파드는 당연히 죽을것이다. 그러나 옵션 설정을 해준다면 일단 파드는 운영할 수 있게 할 수 있다. 
- configmap configMapKeyRef.optional: true configmap이 존재하지 않아도 컨테이너는 시작한다. 


## 하나하나 지정하지 않고 한번에 환경변수 지정하는방법은? 
- 필요한 항목을 각각 지정해준다면 오래걸릴 뿐만아니라 실수할 가능성이 있다. 그렇다면 한번에 전달하는 방법은 있을까? 존재한다. 


```agsl
apiVersion: v1
kind: pod
metadata:
  name: test-pod
spec:
  containers:
  - image: test/test
  envFrom
    configMapRef:
      name: my-config-map
```


## 컨피그맵 항목을 명령줄 인자로 전달하자.
```agsl
apiVersion: v1
kind: pod
metadata:
  name: test-pod
spec:
  containers:
  - image: test/test
  env:
  - name: NUMBER 
    valueFrom:
      configMapKeyRef:
        name: test-config  
        key: NUMBER
  args: [$(NUMBER)]  
```



## volume내에 존재하는 configmap을 파드에 전달하는방법

```agsl
apiVersion: v1
kind: pod
metadata:
  name: test-pod
spec:
  containers:
  - image: test/test
  
  volumeMounts:
  - name; config
    mountPath: /etc/nginx/conf.d  ----> configmap 볼륨을 마운트하는 위치 
    
  volumes:
  - name: config
    configMap:
      name: my-conf  ----> my-conf라는 configmap을 참조한다.
  
```


## 개별 configmap 항목을 마운트하는 방법

```agsl
apiVersion: v1
kind: pod
metadata:
  name: test-pod
spec:
  containers:
  - image: test/test
  
  volumeMounts:
  - name; config
    mountPath: /etc/test/init.conf  디렉토리가 아닌 파일을 마운트 
    subPath: init.conf  ----> 전체 볼륨을 마운트하는 대신 init.conf항목만 마운트  
```
subPath는 모든 종류의 볼륨을 마운트할 때 사용할 수 있다. 전체 볼륨을 마운트하는 대신에 일부만 마운트할 수 있다.
그러나 개별 파일을 마운트하는 이 방법은 파일업데이트와 관련해 큰 결함이 존재한다. 


## configmap 볼륨 안에 있는 파일 권한 설정 
- defaultMode opiton을 사용하자 



## 단일파일에 마운트했을 때 업데이트가 되지 않는다. 
만약 전체 볼륨대신에 단일 파일을 컨테이너에 마운트한 경우에는 업데이트를 해도 업데이트가 되지않는다.
subPath를 사용하는 컨피그맵은 자동으로 업데이트하지 않는다. 
가장 쉬운 방법은 configmap 정보를 업데이트 후 파드를 한번 delete했다가 재 실행 하는것도 하나의 방법이다.