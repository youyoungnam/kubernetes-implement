## 세션 어피니티

### 스티키 세션
- 세션이 유지되도록 하는 것
- 주로 로드 밸런싱 환경에서 세션을 유지하도록 하는것을 말한다.
- istio에는 consistent Hashing이라는 옵션이 있다. 이것을 이용하면 스티키 세션을 할 수 있다. 
  - 예시) A라는 파드와 B라는 파드가 존재한다. Consistent Hashing은 클라이언트에서 들어온 요청이 로드 밸런싱으로 들어왔을 때 이 데이터들을 특정 해싱 알고리즘에 적용해서 출력값을 통해 어떤 파드로 전달할지 구현할 수 있는 옵션이다.
  - 예를들어 클라이언트에서 ip를 가지고 들어올 수 있는데 이 ip가 홀수인지 아니면 짝수인지를 가지고 A파드 혹은 B파드로 갈 수 있게 할 수 있다.
  - 그렇다면 ip를 가지고 홀수 인지 짝수인지를 구분할 수 있다고 한다면, A파드로 갈지 B파드로 갈지 어디서 정하는가? 그냥 vs에 weigth값을 전달하면 되는건가? 그러면 어디가 짝수이고 어디가 홀수인가? 의문점이 많다. 그냥 weigth를 더 많이 주는 쪽이 짝수인것인가..? 이것은 조금 더 알아볼 필요가 있다. 
  - 찾아보니까 이스티오 envoy는 가중치 서브셋과 세션 어피니티를 동시에 사용할 수 없다고 한다. 만약 세션어피니티와 가중치를 같이 사용하게 된다면 이런식으로 동작한다
  - client -----> weight ------> LoadBalancer -----> POD 이미 weight에서 나눠졌는데 결국 로드벨런서는 하나의 파드만 가게되는 것이다. 즉, 로드벨런서에서 두개의 파드로 또 나눠서 가는게 아니다. 
  - 

```
kind: DestinationRule
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: python-service-service-destination
  namespace: default
spec:
  host: python-service # Service
  trafficPolicy:
    loadBalancer:
      consistentHash:
        useSourceIp: true  
  subsets:
    - labels:  # Selector
        version: safe # 파드의 label "safe"
      name: safe
    - labels:
        version: risky # 파드의 label "risky"
      name: risky
``` 