## command와 args 
- command와 args는 파드가 생성된 이후에는 업데이트 할 수 없다. 
- command는 컨테이너 안에서 실행되는 실행 파일
- args는 실행파일에 전달되는 인자 

TIP) ENTRYPOINT - command, CMD - args 


## 컨테이너의 환경변수 설정
- 파드의 각 컨테이너를 우ㅟ한 환경변수 리스트를 지정할 수 있다. 예를들어 A라는 컨테이너가 파이썬 애플리케이션이라고 가정하고 환경변수를 아래처럼 지정한다면 os.environ['INTERVAL']로 불러올 수 있다.
```agsl
apiVersion: v1
kind: Pod
metadata:
  name: appname
  labels:
    name: appname
spec:
  containers:
  - name: appname
    image: gcr.io/google-containers/busybox
    env:
      - name: INTERVAL
        value: "30"
    ports:
      - containerPort: 8080
```

이러한 방식의 단점 파드 정의에 하드코딩된 값을 가져오는 것은 효율적이지만, 프로덕션과 개발을 위해 서로 분리된 파드 정의가 필요하다는 것 여러 환경에서 동일한 파드 정의를 재 사용하려면 파드 정의에서 설정을 분리하는게 좋다. 
configmap을 사용한다면 환경별로 분리하여 사용할 수 있다. 

command line으로 configmap 만들기
```agsl
kubectl create configmap app-config --from-literal=sleep-interval=25

Configmap: app-config
  sleep-interval: 25
  
# 파일을 가지고 configmap만들기
kubectl create app-config --from-file=customkey=config-file.conf
```
경험상 파일을 가지고 configmap만들 때는 나는 주로 사내 IDC 통신에 필요한 사이드카 형태로 만들 때 사용했다. configmap보단 secret으로 만들었다. 


### 볼륨안에 있는 컨피그맵 사용 
- 컨피그맵의 내용을 가진 볼륨을 생성하는 것은 컨피그맵 이름으로 참조하는 볼륨을 만들고 이 볼륨을 컨테이너에 마운트하는 만큼 간단하다. 
```agsl
apiVersion: v1
kind: Pod
metadata:
  name: appname
  labels:
    name: appname
spec:
  containers:
  - name: appname
    image: gcr.io/google-containers/busybox
    ports:
      - containerPort: 8080
    volumeMounts:
      - name: config
        mountPath: /etc/nginx/conf.d # configmap 볼륨 마운트하는 위치
        readOnly: true
  volumes:
    - name: config
      configMap:
        name: ex-config
```


### 디렉토리 안에 다른 파일을 숨기지 않고 개별 컨피그맵 항목을 파일로 마운트
- 컨피그맵의 항목을 개별 파이로 기존 디렉토리 안에 있는 모든 파일을 숨기지 않고 추가하는 방법은 전체 볼륨을 마운트하는 대신 volumeMount에 subPath를 속성으로 파일이나 디렉토리 하나를 볼륨에 마운트할 수 있다.


![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/897a8c6a-4f85-41ab-bf9f-5b3f264c505a)
