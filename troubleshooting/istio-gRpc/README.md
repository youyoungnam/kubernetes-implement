## Istio에서 gRpc 통신이슈 해결하기 

### 이슈 
- 우리 회사 클러스터 내에는 Spring Gateway Application이 하나가 있는데 이 Gateway를 우리 전사 제품을 통과하는 Gateway이다.
- 문제는 이 gateway뒤에 다른 제품 Spring Gateway를 통과하고 backend Application으로 가는 통신이 있다는 것이다

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/85b43b23-63f8-4135-9f4d-557bc0237aa2)


그림을 간단하게 그려보면 이렇게 flow가 흘러 간다는 것이다. 궁금한 점은  이렇게도 gRpc통신이 가능한것인가? spring Gateway가 gRpc를 통신할 수 있는것인가? 다양하게 궁금했다. 

일단 개발환경 클러스터에 gRpc관련 Application올리고 테스트를 해봤더니 역시나 통신이 끊겨버렸다. 여기서 가장 먼저 든 생각은 Spring Gateway가 양방향 통신을 할 수 있게 잡아 주지 않을것이다.
생각을 먼저했다. 그래서 테스트 한 방법이 gRpc 통신 path를 인증을 istio AuthorizationPolicy에서 인증을 통과하게 하지 않고 Spring Gateway 역시 통과하게 하지 않고 바로 gRpc Application으로 통신하게 할 수 있게 만들었다. 
그랬더니 connection은 연결을 성공했다. 


문제는 그 다음인데 gRpc통신하는 다른 서비스들은 통신을 하지 못했다. 로그를 확인해보니 http 1.x 잘못된 request 요청이라고 하는 것이다. 즉 pod to pod에서는 gRpc를 통신을 하지 못하고 있었던 것이다.
다음 해결책은 istio에서 gRpc를 지원하는지도 궁금했다. [Protocol Selection](https://istio.io/latest/docs/ops/configuration/traffic-management/protocol-selection/)문서를 확인해보면 istio sidecar 즉, envoy proxy에서 gRpc를 지원하고 있는거 같다.

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/1cae6d19-6840-4fc5-92cb-0c134fbac61a)

쿠버네티스 resource service name에 grpc를 지정해도 되고 kubernetes 1.18이상이라면 appProtocol: protocol 지정하면된다. gRpc가 필요한 Application service마다 gRpc 옵션을 넣어주니 해결이 되었다. 
