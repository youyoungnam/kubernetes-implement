## istio Virtual Service
- 처음에 이스티오를 사용한다 했을 때 Virtual Service 단어는 너무나 어려워 보였다. 솔직히 어떤식으로 동작하는지도 전혀 몰랐고.. 쿠버네티스도 어려운데 Service Mesh 사용한다고 하니까 어려웠다.

- 단어에 겁을 먹지말고, 내가 생각한 Virtual Service는 직관적으로 라우팅 규칙을 구성을 할 수 있는 것이라고 생각하면된다.


- 이스티오를 사용한다면 Istio-pilot을 알고 있을 것이다 Istio-pilot은 Virtual Service 정의한 yaml를 가지고 있다고 생각하면 된다. 즉 파일럿은 즉시 실시간으로 Envoy Proxy를 구성하는 역할을 한다. 우리가 VS(Virtual Service)에 큰 변동사항이 있다고 가정하면 이러한 정보들이 파일럿으로 가고 파일럿이 사용자 지정 라우팅 규칙의 결과로서 변경되어야 할 모든 프록시에 전달한다.

- 사실 모든 Canary를 배포하는건 envoy이다. 이스티오는 구성이 필요한 모든 프록시를 구성할 능력을 우리에게 주는 것이다. 우리는 사실 프록시에 대해 생각할 필요가 없다. 우리는 단지 높은 수준의 Virtual Service 개념에서 작업할 수 있다. 하지만 실제로 우리가 하는 일은 프록시를 구성하는 것이다.


### VirtualService
```
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: a-set-of-routing-rules-virtual-service # Virtual Service 이름을 작성한다.
  namespace: default
spec:
  hosts:
    - python-service.default.svc.cluster.local #Service DNS 작성해야 한다.
  http:
    - route:
      - destination:
          host: python-service.default.svc.cluster.local 
          subset: safe 
        weight: 100
      - destination:
          host: python-service.default.svc.cluster.local
          subset: risky 
        weight: 0
```



### istio https Virtual Service retries options 문제 직면 해결볍
- 일단 retries options을 사용하게 된 이유는 무엇일까?? 회사 서비스에 대해 자세하게 말할 수 없지만, 우리는 개발환경과 운영환경을 두개의 클러스터를 운영하고 있다. 개발 클러스터에서 배포된 프론트엔드가 다른 프론트엔드 모듈을 호출할 때 503 error를 계속 확인되는 것을 보았다. 이유는 아마도 여러가지 일 수 도 있다. 예를들자면, 권한이라던지, 특정 배포 순간에 서비스를 클릭을 했다던지 여러가지 이유가 있었다.
일단 503 error가 나타나는게 보기 싫었다. Virtual Service에 특정 prefix에 접근 했을 때 에러가 나오면 재시도하는 옵션이 있다. 그게 retries 옵션이다.

```
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: a-set-of-routing-rules-virtual-service # Virtual Service 이름을 작성한다.
  namespace: default
spec:
  hosts:
    - python-service.default.svc.cluster.local #Service DNS 작성해야 한다.
  http:
    - match:
        - uri: # 만약에 prefix가 name이면 python-service로
            prefix: "/name"
      timeout: 30s
      retries:
        attempts: 3
        perTryTimeout: 100ms
        retryOn: gateway-error,connect-failure,refused-stream
    - route: # 그리고 조건이 충족 된다면
        - destination:
          host: python-service.default.svc.cluster.local
```

Retries 옵션에 대한 설명
timeout: 30s: 요청에 대한 최대 대기 시간을 30초로 설정 이는 요청이 완료되기까지 허용되는 최대 시간이다. 만약 요청이 30초 이상 걸릴 경우, 타임아웃으로 간주되고 해당 요청은 실패로 처리된다.

retries: attempts: 3: 요청이 실패할 경우 최대 3번의 재시도를 수행. 따라서, 최초의 요청을 포함하여 총 4번의 시도가 가능.

retries: perTryTimeout: 100ms: 각 재시도의 최대 시간을 100밀리초(0.1초)로 설정. 이는 각 재시도가 최대 0.1초까지 소요될 수 있음을 의미. 따라서, 재시도 시간이 0.1초를 초과하면 해당 재시도는 실패로 처리.

retries: retryOn: gateway-error,connect-failure,refused-stream: 리트라이가 수행될 상태 코드를 지정. 이 경우, 게이트웨이 에러(gateway-error), 연결 실패(connect-failure), 거부된 스트림(refused-stream) 상태 코드가 발생할 때 재시도가 수행될 수 있음 즉, 이러한 상태 코드가 발생하면 Istio는 재시도를 시도할 수 있음.

이러한 설정은 주어진 조건에 따라 최대 3번의 재시도를 수행하고, 각 재시도는 최대 0.1초까지 소요될 수 있다. 또한, 특정 상태 코드에 대한 재시도 조건을 지정하여 해당 상태 코드가 발생할 경우 재시도를 수행할 수 있도록 구성되었다.


### 문제
- 그렇다면 어떤문제가 있었을까? 우리는 개발환경 클러스터에서 운영환경 클러스터를 옮기는 작업을 진행중이였다. 모든 사람들이 알고 있겠지만, 운영환경은 https를 적용해야한다. 그래서 개발환경에서 사용되는 인증권한 서비스 등등 관련된 서비스에 https를 다 적용해야했다. 그렇다보니 알 수 없는 에러들이 나왔다. 이스티오에 ssl를 적용하는 것도 처음이라 쉽지 않았다. 솔직히 제대로 된지도 몰랐는데 잘 찾아보고 문서를 읽다보니 적용을 했다. 그 다음이 문제다.
- 인증/권한 관리 및 oauth service사이 문제 등등 생기기 시작했다. 인증/권한 관리 서비스에서 ssl 적용하고 VS를 retries옵션을 개발환경에서 적용했던 것을 그대로 적용하다보니 문제가 생겼던것 이였다. 계정을 처음 생성했는데 이미 존재하는 계정이라고 에러가 발생 한거였다. 알고보니 인증/권한 관리에 ssl를 적용하다보니 retries옵션이 작용했던 것이다.
- retries 옵션 설명을 가져와 보면 retries: Retry policy for HTTP requests. http 요청일 때 작용하는 것이다. 즉 우리는 ssl를 적용시키고 retries를 넣어버리니 한번에 5번씩이나 요청이 들어갔던것이다. 이것을 oauth에도 적용하니 이미 발급된 토큰인데도 불구하고 5번씩 계속 요청하니 500 server error를 만난것이다. 
- 이 문제를 3일동안 잡고있다가 결국 해결했다. 