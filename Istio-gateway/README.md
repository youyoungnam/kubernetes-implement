## istio-gateway
- 만약에 파드안에 있는 container가 다른 파드에 있는 container를 호출한다면, envoy로 붙어있는 사이드카 프록시를 통해 전달된다. 그리고 우리는 이러한 프록시를 라우팅 규칙으로 관리를 할 수 있다. 
- 사용자가 만약에 youngnam.com이라는 사이트를 검색해서 들어온다고 가정해보자. 그런데 youngnam.com은 클러스터 내부에 배포가 되어 있다. 여기서 youngnam.com 개발자들이 특정 기능을 개발을 했고 그것을 이미지를 만들고 레지스트리에 올렸다고 가정해보자. 
- 그리고 youngnam app이 NodePort로 외부에서 접근할 수 있다고 하자. 현재 youngnam.com은 클러스터에 배포되어 있고 이스티오를 사용중이라고도 하자. 
- youngnam-app ---> original, youngnam-app ---> test 이렇게 카나리아 배포를 해서 기능을 테스트 하려고 하는데 실제로 되지 않는다. 왜일까?? 
- 이스티오 Service Mesh를 사용중인것이면 모든 네임스페이스에 istio-injection=enabled 되어 있을것이고 모든 파드에 사이드카 형태로 붙어서 통신을 할것이다. 근데 여기서 중요한건 yougnam-app original과 test 두개의 파드도 마찬가지로 이스티오 프록시 형태로 통신하고 있다는 것이다. 여기서 virtual service 와 destination-rule를 적용해도 먹지가 않는다는 것이다. 
- 클러스터 맨앞단에 진입할 때 프록시로 전달해 주는 파드가 없어서 그런것이다. 즉, istio-ingress-gateway가 없어서 그런것이다. 해당 파드 서비스는 nodePort로 개방되어 있기때문에 그냥 바로 서비스로 접근하는 것일 뿐인것이다. 
- 즉 이스티오 게이트웨이를 맨 앞단에 두고 이 프록시가 virtual service 와 destination rule이 작동할 수 있게 적용을 해줘야 한다는 것
- 쉽게 생각해보면 이스티오 게이트웨이는 일종의 프록시 역할을 하는 것이다. 


