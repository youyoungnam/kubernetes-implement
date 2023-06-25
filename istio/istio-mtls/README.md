# 클러스터 내부 암호화가 중요한 이유 
- 사실 우리 쿠버네티스 클러스터를 보면 노드들의 집합으로 이루워져 있다. 쿠버네티스 리소스들이 노드에 배포되어 있는것이다. 우리가 클라우드 서비스를 사용해서 쿠버네티스 클러스터를 구성하게 되면 region을 선택할 수 있는데 이 region이 같은 위치에 있으면 괜찮은데 만약 다른 데이터센터에 노드가 배치된다면 과연 안전할까?
  ![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/e19a3e28-366f-4c48-b07a-8478da25f4a8)
- 만약에 다른 노드 즉 다른 데이터센터에 있는 노드에서 다른 데이터센터에 위치한 파드에 호출한다면 http통신을 통해서 서로 통신하게 될것인데 그렇다면 이건 외부로 http를 통신하게 되는 것이고 만약 해커가 중간에 데이터를 쉽게 가져갈 수 있을 것이다. 

- 정리하자면, 우리 클러스터는 다중 노드로 이어져 있고 그 노드마다 파드들이 배포되어 있다. 특정 노드에 배포된 파드내에 컨테이너가 다른 노드에 있는 파드에 있는 컨테이너에게 요청을 하게 되면 이 호출은 http로 통신을 하게되고 결국 이것을 통제할 필요가 있다. 

## istio-citadel
- citadel은 이스티오가 통신할 때 TLS(SSL)암호화를 진행하여 통신을 할 수 있다. citadel은 암호화 또는 사용자인증에 필요한 인증서 관리를 해준다. 


## auto mtls 활성화
- pod <----> pod 트래픽으로 tls를 활성화 하는 것이 중요하다. 우리는 istio를 사용하고 있기 때문이 이부분은 걱정안해도 된다. 왜냐하면 istio는 기본적으로 mtls가 활성화 되어 있기 때문이다.

## Permissive mTLS 
- 한쪽 Pod는 sideCar가 주입되어 있지 않는 Pod 다른 한쪽은 sideCar가 주입되어 있는 Pod가 있다고 가정하자. 만약에 envoy Proxy가 없는 파드에서 있는 파드를 호출하게 된다면 tls가 설정되지 않는 상태로 통신하게 될것이다. tls가 없다는것은 보안이되지 않는 상태로 통신을 하게되는 것이다.
![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/c2cc5413-16e8-455f-a972-9254e6faa0ec)



## Strict mTLS
- 한쪽 Pod는 sideCar가 주입되어 있지 않는 Pod 다른 한쪽은 sideCar가 주입되어 있는 Pod가 있다고 가정하자. 만약 주입되어 있는 파드에서 envoy sidecar가 주입되어 있지 않는 파드를 호출하게 된다면 호출되지 않는다. 즉 연결거부상태로 된다.
![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/149599bd-5351-4a0e-a44f-fe87a111efe6)
