#  Nginx Ingress Controller + istio 연결 이슈 


## Ingress 
- 클러스터 외부에서 내부 서비스로 접근하는 HTTP, HTTPS 요청을 어떻게 처리할지 정의
- Ingress를 사용하지 않았을 때 쿠버네티스가 외부 요청을 처리할 수 있는 방법은 NodePort, ExternalIP 방법이 있다. 
  - NodePort, ExternalIP는 Layer 4 (TCP, UDP) 해당되기 때문에 네트워크에 대한 처리 요청이 한계가 있다.

## Ingress Controller 
- Ingress Controller는 Ingress가 동작하기 위해 반드시 필요한 Controller다.
- Istio Ingress는 istio 기반 Ingress Controller다.
- Ingress 정의 후 Ingress Controller에 적용함으로 써 추상화 단계에서 서비스로직을 처리할 수 있다. 

```agsl
kind: Ingress
metadata:
  annotations:
      "nginx.ingress.kubernetes.io/ssl-redirect":"true"
      
spec:
  rules:
  - hosts: "DNS ex)yooyoungnam.com" 1번 
  http: 2번
    paths:
    - path: / 3번
      pathType: Prefix
    - backend:
        service:
        name: istio-ingress 4번
        port:
          number: 80
```

## Ingress 처리과정
- yooyoungnam.com이라는 호스트 명으로 접근하는 네트워크 요청에 대해서 Ingress 규칙을 적용
- http 프로토콜을 통해 / 라는 경로로 접근하는 요청을 istio-ingress라는 서비스 이름의 80 포트로 전달

## 중요한 점
- Ingress만 정의하고 배포한다고 해서 Ingress가 작동하는것이 아니다 위에서 말했듯이 Ingress Controller가 존재해야한다.
- Ingress Controller가 외부로부터 네트워크 요청을 수신했을 때, Ingress 규칙에 기반해 이 요청을 어떻게 처리할지를 결정


## 생겼던 이슈 
- Ingress에서 백엔드 Service name을 잘못 바라보고 있었음 그렇다보니 아무리 접근해도 정상으로 작동안했음
- istio service 네임을 제대로 설정해야한다. label 확인 필요 

## Flow
![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/72c8fb75-b604-4405-ba15-71b8773d1e85)



