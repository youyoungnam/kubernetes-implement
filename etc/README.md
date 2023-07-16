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




### configmap과 secret 둘중 어느것을 사용해야할까
- 민강하지않고 일반 설정 데이터는 configmap을 사용
- 본질적으로는 민감한 데이터는 시크릿을 사용해 키 아래에 보관하는게 필요
- 만약에 설정파일에 민감한 데이터와 민감하지 않은 데이터가 동시에 있다면 시크릿을 사용하는게 좋다.



시크릿은 컨피그맵과 다르게 yaml 내부를 확인해 보면 바이너리 데이터로 되어 있는것을 확인할 수 있다. 시크릿은 바이너리 데이터도 담을 수 있다. 하지만 시크릿의 최대 크기는 1MB이다.


만약에 특정 파드가 tls인증서를 사용하는지 안하는지 테스트하고 싶다면 아래 방법을 통해 확인할 수 있다.
```agsl
kubectl port-forward pod-name 8443:8443 & curl https://localhost:8443 -k
```


secret 볼륨은 시크릿 파일을 저장하는 데 인메모리 파일시스템(tmpfs)을 사용한다. 컨테이너에 마운트된 볼륨을 조회하면 알 수 있다.

TIP) 쿠버네티스에서는 시크릿을 환경변수로 노출할 수 있게 해주지만, 이 기능을 사용하는 것이 가장 좋은 방법은 아니다. 애플리케이션은 일반적으로 error를 보여줄 때 환경변수를 기록하거나 시작하면서 로그에 환경변수를 의도치않게 남길 수 있다
그리고 자식 프로세스는 상위 프로세스의 모든 환경변수를 상속받는다 그 때 애플리케이션이 타사 바이너리를 실행할 경우 시크릿 데이터를 어떻게 사용하는지 알 방법이 없다. 
즉, 환경변수로 시크릿을 컨테이너에 전달하는 것은 의도치 않게 노출될 수 있다. 그렇기 때문에 생각하고 사용해야한다. 그렇다면 어떻게 사용해야할까? 시크릿 볼륨을 통해 사용해야 한다.
```agsl
env:
- name: TEST_SECRET
  valueFrom:
    secretKeyRef:
       name: test-https
       key: yy
```

### DownLoad API 
download API는 파드의 레이블이나 어노테이션과 같이 어딘가에 이미 설정된 데이터를 여러곳에 반복해서 설정하고 싶지않을 때 사용하면좋다. 즉, 환경변수 또는 download API volume 내에 잇는 파일로 파드와 해당 환경의 메타 데이터를 전달할 수 있다. 
Download API는 애플리케이션이 호출해서 데이터를 가져오는 REST 엔드포인트와 다르다.

download API를 사용하면 아래 항목들을 컨테이너에 전달할 수 있다.
- 파드의 이름
- 파드의 ip주소
- 파드가 속한 네임스페이스
- 파드가 실행 중인 노드의 이름
- 파드가 실행 중인 서비스 어카운트 이름
- 각 컨테이너의 cpu와 메모리 요청
- 각 컨테이너의 cpu와 메모리 제한
- 파드의 레이블
- 파드의 어노테이션

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
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 8080
    env:
      - name: POD_NAME
        valueFrom:
          fieldRef:
            fieldPath: metadata.name
      - name: POD_NAMESPACE
        valueFrom:
          fieldRef:
            fieldPath: metadata.namespace
      - name: POD_IP
        valueFrom:
          fieldRef:
            fieldPath: status.podIP
      - name: NODE_NAME
        valueFrom:
          fieldRef:
            fieldPath: spec.nodeName
      - name: SERVICE_ACCOUNT
        valueFrom:
          fieldRef:
            fieldPath: spec.serviceAccountName
```
파드가 실행되면 파드 스펙에 정의한 모든 환경변수를 조회할 수 있다.


## 쿠버네티스 API 서버와 통신하기 
- Download API는 단지 파드 자체의 메타데이터와 모든 파드의 데이터 중 일부만 노출한다. 떄로는 애플리케이션에서 클러스터에 정의된 다른 파드나 리소스에 관한 더 많은 정보가 필요할 수도 있다. 이경우는 download api는 도움되지 않는다.


## 쿠버네티스 API
- kubectl cluster-info를 통해 api서버 주소를 알아낼 수 있다.
```agsl
kubectl cluster-info
```
사실 서버는 HTTPS를 사용중이라 인증이 필요하기 때문에 직접 통신하는 것은 간단하지않다. curl를 통해 접근하고 -k옵션을 사용해서 서버 인증서 확인을 건너뛰도록 할 수 있지만 원하는 결과는 얻을 수 없다. 


```agsl
curl https://192.168.99.100:8443 -k
```

## kubectl proxy를 통해서 API 서버 액세스하는 법
kubectl proxy명령어를 통해 로컬 컴퓨터에서 HTTP 연결을 수신하고 이 연결을 인증을 관리하면서 api 서버로 전달하기 때문에 요청할 때마다 인증 토큰을 전달할 필요는 없다.

```agsl
kubectl proxy
starting to serve on 127.0.0.1:8001

 curl localhost:8001
{
  "paths": [
    "/.well-known/openid-configuration",
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    "/apis/admissionregistration.k8s.io",
    "/apis/admissionregistration.k8s.io/v1",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1",
    "/apis/apiregistration.k8s.io",
    "/apis/apiregistration.k8s.io/v1",
    "/apis/apps",
    ....
  ]
}%
```

## 파드 내에서 API 서버와 통신하기
kubectl proxy를 사용해서 로컬 컴퓨터에서 api 서버와 통신하는 방법을 알았는데 그렇다면 kubectl이 없는 pod내에서는 어떤식으로 해야할까? 파드 내에서 api 서버와 통신하는 방법은 3가지를 먼저 처리해야한다
 - api 서버 위치알기
 - api 서버와 통신하고 있는지 확인
 - api 서버로 인증해야한다.


curl을 사용할 수 있는 환경이여야 하기 때문에 바이너리가 포함된 컨테이너 이미지를 사용하자. 도커 허브에서 tutum/curl 이미지를 사용하자 

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
    image: tutum/curl
    command: ["sleep", "999999"] # container가 계속 실행되도록 하려고 지연시간이 길게 sleep할 수 있게 만들어야 한다.
```

파드를 만든 후 kubectl exec를 실행하여 컨테이너 내부에서 bash shell을 실행해보자 
```agsl
kubectl exec -it curl bash
```
api 서버의 ip주소와 포트를 컨테이너 내부의 kubernetes_service_host, kubernetes_service_port변수를 얻을 수 있는지 확인

```agsl
env | grep KUBERNETES_SERVICE
```

api서버는 https의 기본 포트인 443에서 수신 대기 중이므로 https로 접속할 수 있다. curl https://kubernetes 해당 주소롤 curl 사용한다면 인증하라는 메세지를 확인할 수 있다.
이 문제를 간단히 해결하는 방법은 -k 옵션을 사용해서 접근하면 해결할 수 있다. 하지만, 실제 애플리케이션에서 사용할 때는 인증을 건너뛰면 안된다. 

API 서버로 인증
```agsl
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

curl -H "Authorization: Bearer $TOKEN" https://kubernetes
```

파드가 실행중인 네임스페이스 얻기

```agsl
NS=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)

curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespace/$NS/pods
```
같은 방식으로 다른 API 오브젝트를 검색하고 간단한 GET요청 대신 PATCH를 전송해 업데이트할 수 있다. 



## 정리 
- Application은 API 서버의 인증서가 인증 기관으로부터 서명됐는지 검증해야 한다. 인증 기관의 인증서는 ca.cart파일에 있다.
- 애플리케이션은 token 파일의 내용을 Authorization HTTP 헤더에 Bearer 토큰으로 넣어 전송해서 자신을 인증해야 한다.

  ![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/0d1747fb-1818-415a-87eb-b42e8119e216)
