## Circuit Breaking
- 쿠버네티스에 파드들이 올라가게 되면 이러한 파드들은 서비스들로 연결되어 있다. 만약에 어떤 이유로 마이크로 서비스에 문제가 생겼다고 한다면 이 문제는 네트워크 문제일 수 도 있고 아니면 컨테이너 내부 문제일 수 도 있고 아니면 잘못된 이미지를 가져와서 파드가 생성이 안될 수 도 있다. 
- 예를들어 A라는 마이크로 서비스에서 B라는 마이크로 서비스에 통신이 느려졌다면 어떻게될까? 실패로 이어질 가능성이 높다. 연계고장(Cascading failure)은 예측이 불가하다. 어떠한 문제가 생길지 모른다. 그렇다면 연계고장을 피할 수 있을까? 연계고장에 대한 대비는 서킷 브레이킹이라는 패턴이 존재한다. 
- istio-Document에 작성되어 있는 것을 가져와본다면
  Circuit breaking is an important pattern for creating resilient microservice applications. Circuit breaking allows you to write applications that limit the impact of failures, latency spikes, and other undesirable effects of network peculiarities.

총 쿠버네티스에 파드가 3개가 올라와있다고 가정하자. A라는 파드, B라는 파드, C라는 파드가 존재한다고 했을 때 이러한 파드들은 서비스들이 서로 연결되어 있다. 여기서 기억해야할 것은 B라는 파드가 서비스에 문제있다고 가정하는 것이다. 즉 A라는 파드에서 B라는 파드에 요청을 했을 때 요청시간이 30초 이상이 걸리고 실패한다고 했을 때 서킷 브레이킹 로직은 이러한 것들을 모니터링을 할 수 있다. 
그리고 서킷브레이킹은 이러한 모든 요청 전달을 멈추게 할 수 있다는 것이다. 즉 마이크로 서비스와 연결을 효과적으로 차단할 수 있는 것이고 그 시간동안 서비스를 다시 복구 할 수 있는 시간을 주는 것이다. 


## istio-Circuit Breaking
- istio에서 circuit breaking을 설정하는 곳은 Destinationrule에서 적용하면된다. 
- 옵션
  - maxEjectionPercent 백분율로 표시 만약에 서킷 브레이커가 작동한다면 부하분산을 백분율로 제거한다는 의미 
  - consecutiveErrors host가 connection pool에서 제외되는 오류수 default 5  
  - interval 특정 시간동안 연속적으로 오류가 발생한다면 서킷 브레이커가 트리거를 한다는 옵션
  - baseEjectionTime circuit break 시간 default 30s