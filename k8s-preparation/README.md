# 쿠버네티스와 같은 시스템이 필요한 이유
- 예전에는 애플리케이션이 거대한 모놀리스 방식으로 개발되었다. 즉, 이러한 거대한 모놀리스 애플리케이션이 작은 마이크로 서비스로 세분화함과 동시에 해당 애플리케이션을 실행하는 인프라의 변경으로 인한 것이다. 

## 거대한 애플리케이션에서 마이크로서비스로 전환
- 모놀리스 애플리케이션은 모든것이 강결합 되어있다. 모든것이 강결합 되어있다는건 어떤 의미일까? 예를들어 어떤 인증 컴포넌트가 있다고 가정해보자 이러한 인증 컴포넌트는 다른곳에서도 사용중하고 있을 가능성이 높다 만약 다른 기능이 변경된다면 인증 컴포넌트도 변경될 수 도 있다. 이런식으로 결합이되어 있다는 의미로 알고 있으면 될거 같다. 그리고 인증부부만 변경 했는데 다른부분까지 다 한번에 재 배포를 해야하는 상황이 온다. 이런식으로 개발을 진행한다면 상호의존성의 제약이 커지고 전체 시스템의 복잡도가 높아질 것이다. 
- 이러한 복잡한 모놀리스 애플리케이션을 마이크로 서비스라는 독립적으로 배포할 수 있는 작은 구성 요소로 분할해야 한다. 각 마이크로 서비스는 독립적인 프로세스로 실행되며 단순하고 잘 정의돈 인터페이스로 다른 마이크로 서비스와 통신한다.
  - 생각해보면 회사에서도 쿠버네티스로 서비스를 운영중이다. 몸으로 느끼는 단점은 각각에 대한 애플리케이션에 대한 CI/CD를 구축해야한다는 점이다. 즉 애플리케이션이 추가될 때 마다 CI/CD가 추가된다는 점이다. 모놀리스 서비스는 하나의 CI/CD만 구축하면 되는데 마이크로 서비스는 각각에 대한 CI/CD를 구축해야 한다는 점이 존재한다. 
  - 그리고 뭔가 서비스 처럼 돌아가게 만들려면 누군가는 서비스에 대해 제대로 알고 있어야 한다는 점이다. 예를들어 프론트엔드를 마이크로 서비스로 구축했다면, 메인 서비스에서 다른 마이크로 서비스들을 호출 할 수 있게 엔드포인트 설정을 해줘야 할 것이다. 
  - 개발 이외에도 협업 즉, 소통이 중요하다. 사람들은 자기만의 성격이나 생각을 가지고 있기 때문에 여러가지 문제가 생길 수 있다..
  

## 컨테이너
- 컨테이너 내부를 알고 사용하자. 컨테이너가 동작할 수 있는 이유는 첫 번째는 리눅스 네임스페이스로 각 프로세스가 시스템에 대한 독립된 뷰를 볼 수 있어서다.그렇다면 네임스페이스란 무엇일까?
  - 마운트(mnt)
  - 프로세스ID(pid)
  - 네트워크(net)
  - 프로세스 간 통신(ipc)
  - 호스트와 도메인 이름(uts)
  - 사용자 ID(users)
- 기본적으로 리눅스 시스템은 초기 시작할 때 하나의 네임스페이스가 존재하고 파일시스템, 프로세스, 사용자 네트워크 인터페이스 등과 같은 모든 시스템 리소스는 하나의 네임스페이스에 속한다. 프로세스는 동일한 네임스페이스 내에 있는 리소스만 볼 수 있다. 
- 두 번째는 Cgroups 프로세스가 사용할 수 있는 리소스의 양을 제한할 수 있다.
  - cpu
  - memory
  - network

## 도커 컨테이너 등장
- 컨테이너 기술은 오랫동안 사용돼 왔지만, 도커 칸테이너가 등장으로 널리 알려지게 된것이다. 여러 시스템에 쉽게 이식이 가능하게 하는 최초의 시스템인것이다. 
- 애플리케이션 배포에 필요한 라이브러리, 종속성 등등 도커를 실행하는 다른 컴퓨터에 애플리케이션을 배포하는데 사용할 수 있는 것이다. 


## 도커 이미지 레이어 
- 도커 이미지는 레이어로 구성돼어 있다. 모든 도커 이미지는 다른 이미지 위에 빌드되고 두 개의 다른 이미지는 기본 이미지로 동일한 부모 이미지를 사용할 수 있다. 다른 이미지에는 정확히 동일한 레이어가 포함될 수 있다. 
  - 첫 번째 이미지의 일부로 전송한 레이어를 다른 이미지를 전송할 때 다시 전송할 필요가 없기 때문에 네트워크로 이미지를 배포하는 속도가 빨라진다. 
- 도커 이미지 레이어는 배포를 효율적으로 할 뿐만 아니라 이미지의 스토리지 공간을 줄이는 데 도움된다. 각 레이어는 동일 호스트에 한 번만 저장된다. 
- 주의할 점 도커는 도커 자체가 프로세스를 격리하는게 아니다. 컨테이너 격리는 리눅스의 네임스페이스 기술과 cgroup과 같은 커널 수준에서 수행되는 것이다.


## 쿠버네티스 등장
- 쿠버네티스는 오랜 세월 동안 구글 보그 --> 오메가로 변경된 시스템이라는 내부 시스템을 개발해 애플리케이션 개발자와 시스템 관리자가 수천 개의 애플리케이션과 서비스를 관리하는 데 도움을 줬다. 수십만 대의 머신을 우녕할 때, 사용률이 조그만 향상돼도 수백만 달러가 절약되므로 이러한 시스템을 개발할 동기는 분명하다. 구글은 10년동안 보그와 오메가를 비밀로 유지하고 2014년 보그 오메가 내부 구글 시스템으로 얻은 경험 기반으로 쿠버네티스를 출시했다. 
  - 아마도 구글같은 큰 기업은 서비스가 적어도 수천개의 컨테이너가 존재했을 거다. 이러한 컨테이너 운영하는데 힘든점이 있을 가능성이 높았을거 같다. 진짜 상상도 안된다. 그냥 도커 컨테이너 10개만 해도 운영하기가 힘들텐데 이런게 수천개가 있다고 생각하니 끔찍하다. 
  - 구글은 아마도 내부에서는 쿠버네티스 이상의 기술을 가지고 있을거 같다.


## 쿠버네티스가 Yaml(디스크립션) 실행하는 방법
- kubernetes api 서버가 애플리케이션 디스크립션을 처리할 때 스케줄러(컨트롤 플레인 포함되어 있음) 각 컨테이너에 필요한 리소스를 계산하고 해당 시점에 각 노드에 할당되지 않은 리소스를 기반으로 사용 가능한 Worker Node에 지정된 컨테이너를 할당한다 그런다음 kubelet(워커노드에 존재)은 컨테이너 런타임에 필요한 컨테이너 이미지를 가져와서 컨테이너를 실행하도록 지시한다. 
  - 만약에 하나의 파드에 두개의 컨테이너가 있다면 해당 컨테이너들은 같은 노드에 배치될것이다. 이 컨테이너들이 따로 배치되서는 안된다.


![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/4b9a982d-9fc0-437b-8466-42ab98b32740)




## 쿠버네티스 아키텍처 
- 쿠버네티스 클러스터를 이루는 구성 요소는 컨트롤플레인과 (워커)노드로 이루워져 있다.


## 컨트롤플레인 구성 요소
- etcd 분산 저장 스토리지
- API 서버 
- 스케줄러
- 컨트롤러 매니저 
4가지 구성요소들은 클러스터 상태를 저장하고 관리하지만, Application을 실행하는건 아니다.

## 워커노드 구성요소
- Kubelet
- 쿠버네티스 서비스 프록시(kube-proxy)
- 컨테이너 런타임(Docker, rkt...)


![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/7710fb1d-e6da-47f6-a6c8-3c6eec045f14)

위에서 언급한 컨틀롤러 구성요소나 워커노드 구성요소들은 개별 프로세스로 실행되는 것이다. 

알아야할 점 쿠버네티스 시스템 요소는 오직 API Server와 만 통신을 한다 위 그림처럼 다른 구성요소들은 ectd와 직접 통신할 수 없고 api server와 통신을 진행한다. 

쿠버네티스에서 다 중요하지만 특별히 중요한 점은 etcd 와 api server다. etcd와 api server는 여러 인스턴스에 할성화해서 작업을 병렬로 처리할 수 있다. 



TIP)kubelet은 무조건 시스템 구성요소(Daemon)로 실행해야한다.



## 쿠버네티스는 etcd를 어떻게 사용할까
- 우리가 애플리케이션을 파드로 배포하던 deployment로 통해 배포를 하던 아니면 다양한 쿠버네티스 resouces를 사용해 애플리케이션을 배포하게 되면 이 배포가 실패를 하던지 성공을 하던지 사용한 yaml 파일은 ectd key value형태로 저장이된다. 
- 위에서 말했던것처럼 etcd는 오로지 api server와 통신을 한다. 클러스터 상태와 메타데이터를 저장하는 저장하는 유일한 장소가 etcd이다.


## 낙관적 동시성 제어
-  데이터를 잠금을 설정해 그동안 데이터를 읽거나 업데이트를하지 못하도록 하는 대신 데이터에 버전 번호를 포함하는 방법이다.
업데이트 될 때 마다 버전 번호는 증가 클라이언트가 데이터를 읽은 시간과 업데이트를 제풀하는 시간 사이에 버전 번호가 증가했는지 여부를 체크 


## 리소스를 etcd에 저장하는 방법
- etcd의 각 키는 다른 키를 포함하는 디렉토리이거나 해당 값을 가진 일반 키다. 쿠버네티스는 모든 데이터를 /registry아래에 저장한다. 
  - /registry/configmap
  - /registry/daemonsets
  - /registry/events
  - /registry/namespace
  - ...

쿠버네티스는 다른 모든 구성 요소가 api server를 통신하도록 만들어졌다. api server에 낙관성 동시성 제어 메커니즘을 구현해서 클러스터의 상태를 업데이트하기 전에 오류가 발생할 가능성을 줄이고 항상 일관성을 유지할 수 있다. 



### 우리가 자주 사용하는 명령어 kubectl ... 내부에서는 많은 일들이

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/36092a5b-10fc-490b-be16-a3ce60a702e2)

api Server는 이보다 더 많은 기능들이 있게지만 짧게 표현을 했다. 
1. 인증 플러그인
   - 사용자를 클라이언트 인증서 혹은 http 헤더에서 가져온다. 플러그인은 클라언트의 사용자 이름, 사용자 ID, 속해있는 그룹정보
그렇다면 우리가 사용하고 있는 GKE에서는 kubectl 명령어를 사용하게 된다면 어떤식으로 흘러가는지 보자 


![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/45ff121f-b2f5-46fd-b698-5538abb242c6)

그림에 나오는것처럼 user는 아마도 gcp에서 관리하는 serviceAccount 이면서 iam에는 권한이 있을거같다. 예를들자면 클러스터 관리자 권한 or 클러스터 뷰어 권한처럼 말이다. 



## 스케줄러
- 스케줄러는 api server의 감시 메커니즘을 통해 새로 생성될 파드를 기다리고 있다가 할당된 노드가 없는 새로운 파드를 노드에 할당하기만 한다. 
- 스케줄러는 api server로 파드 정의를 갱신하는 것 뿐이다. 
- api server는 감시 메커니즘을 통해 kubelet에 파드가 스케줄링된 것을 통보한다. 대상 노드의 kubelet은 파드가 해당 노드에 스케줄링 된것을 확인하자마자 파드의 컨테이너를 생성하고 실행한다.
- 스케줄러는 따로 정리를 해야할거 같다 많은 내용이 있기때문에... 

TIP Application을 배포할 때 파드 내에 schedulerName이라는 옵션을 사용해서 사용할 파드를 스케줄링할 때 사용할 스케줄러를 지정할 수 있다. 
이말은 즉, 사용자가 직접 스케줄러를 구현해서 클러스터에 배포할 수 있고 다른 설정 옵션을 가진 쿠버네티스의 스케줄러를 배포할 수 있다는 것 


## 컨트롤러 매니저
- 컨트롤러는 다양한 작업을 수행하지만 모두 api 서버에서 리소스(deployment, pod, service ...)변경되는 것을 감시하고 각 변경 작업(새로운 오브젝트를 생성하거나 이미 있는 오브젝트의 갱신 혹은 삭제)을 수행한다.
- 컨트롤러 역시 api 서버에 연결하고 감시 메커니즘을 통해 컨트롤러가 담당하는 리소스 유형에서 변경이 발생하면 통보해줄 것을 요청한다. 


## Kubelet
- kubelet은 워커 노드에서 실행하는 모든 것을 담당하는 요소 
- kubelet이 실행중인 노드를 노드 리소스로 만들어서 api 서버에 등록하는 것
- kubelet은 api 서버를 지속적으로 모니터링해 해당 노드에 파드가 스케줄링되면 파드의 컨테이너를 시작
- 실행중인 컨테이너를 계속 모니터링하고 상태 체크하며 이벤트 및 리소스 사용량을 api서버에 보낸다. 


## kube-proxy
- kube-proxy는 서비스의 ip와 포트로 들어온 접속을 서비스(혹은 파드가 없는 서비스 엔드포인트)를 지원하는 파드 중 하나와 연결시킨다. 
- 서비스가 2개 이상의 파드를 지원하는 경우 프록시는 파드 간에 로드 밸런싱을 수행한다. 


## 전체적인 deployment 생성과정

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/56465444-8a72-49db-a474-50d3c896d63c)


## 파드의 네트워크
- 파드의 모든 컨테이너가 동일한 네트워크와 리눅스 네임스페이스를 공유하는 법은 퍼즈(pause) 컨테이너가 같이 배포되기 때문이다.
- 즉, 한 파드내에 여러개 컨테이너가 있다고 했을 때 결과적으로 동일한 리눅스 네임스페이스를 공유하는 3개의 컨테이너를 실행하는 것


## 네트워크 동작방식
![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/8fc6525c-8603-4b26-88d3-99b53b2d2377)

노드의 파드는 가상 이더넷 인터페이스 쌍을 통해 같은 브리지로 연결된다. 

- 컨테이너가 시작되기 전에, 컨테이너를 위한 가상 이더넷 인터페이스 쌍(veth)가 생성된다. 노드에서 ifconfig를 command실행하면 veth가 남아있는걸 확인할 수 있다. 
- 호스트의 네트워크 네임스페이스에 있는 인터페이스는 컨테이너 런타임이 사용할 수 있도록 설정된 네트워크 브리지에 연결된다. 
- 컨테이너 eth0 인터페이스는 브리지의 주소 범위 안에서 ip할당 받는다. 
- 컨테이너 내부에서 실행되는 애플리케이션은 eth0인터페이스(컨테이너 네임스페이스의 인터페이스)로 전송하면 호스트 네임스페이스의 다른 쪽 veth 인터페이스로 나와 브리지로 전달된다. 
- 이의미는 브리지에 연결된 모든 네트워크 인터페이스에서 수신할 수 있다는 것을 의미한다. 

## kube-proxy가 iptables를 다루는 법
- api server에서 서비스를 생성하게 된다면, 가상 ip 주소를 만든다. 바로 api 서버는 워커 노드에서 실행 중인 모든 kube-proxy 에이전트에 새로운 서비스가 생성되었다고 알린다. 
- 각 kube-proxy는 실행중인 노드에 해당 서비스 주소로 접근할 수 있도록 만든다. 
