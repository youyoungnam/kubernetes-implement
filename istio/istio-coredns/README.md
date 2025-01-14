# Istio DNS Proxy Cache Core DNS 개선하기 


## 소개
### Kubernetes의 CoreDNS의 역할
- CoreDNS는 경량화된 설계로 클라우드 네이티브 환경에 최적화되어 있습니다.
- 플러그인 기반으로 다양한 기능을 추가할 수 있어 고도로 확장 가능합니다.
- **Pod-to-Pod 통신**: CoreDNS를 통해 IP 주소 대신 **DNS 이름**을 사용하여 다른 서비스와 통신할 수 있습니다.
- **DNS 규칙**: Kubernetes 네임스페이스 및 서비스 이름 규칙에 따라 요청을 처리합니다.
- **가상 네트워크 관리**: Kubernetes 클러스터 내에서 네트워크 요청을 효율적으로 처리합니다.


### 위에서 말하는 DNS 규칙이 무엇일까요?
**서비스 디스커버리**
- 서비스 디스커버리를 짧게 설명해보자면, 쿠버네티스 내에 배포된 서비스들이 존재하고, 해당 서비스들을 통신을 가능하게 하는 중요한 메커니즘 입니다.
- 서비스 디스커버리는 애플리케이션(POD)가 동적으로 생성되고 종료되더라도 안정적이고 일관된 방식으로 통신을 지원합니다. 
- 즉, pod는 pod-cidr를 통해 ip를 할당 받고 파드가 죽어서 다른 ip를 할당 받게된다면, 이를 일관된 방식으로 통신을 할 수 있게 해줄 수 있게 해주는 방식입니다. 


### Pod가 다른 서비스인 Pod를 찾는 방법

![img.png](img.png)


간단하게 그림을 통해 설명해보자면
1. Worker2번에 올라가 있는 Pod가 Worker1번에 있는 Nginx Pod를 찾을 수 있는 방법은 첫번째로 APP파드가 CoreDNS에게 nginx.default.svc.cluster.local로 질의를 합니다.
  2. CoreDNS는 해당 DNS(FQDN)에 해당하는 service ip를 다시 APP에게 전달합니다.
  3. APP은 service ip를 가지고 nginx와 통신을 하게 됩니다. 

실제로는 kube-proxy, iptables, CNI 등 더 많은 컴포넌트들이 관여하는 복잡한 과정이 있습니다. 


### Istio가 어떻게 CoreDNS 부하를 감소시킬 수 있을까요?

![img_1.png](img_1.png)https://istio.io/latest/blog/2020/dns-proxy/#automatic-vip-allocation-where-possible

- Istio는 기존에 많이 알려진것처럼, Service Mesh입니다. 그렇다면, Service Mesh는 무엇일까요? Service Mesh는 공식문서를 보면 가시성과 트래픽관리 그리고 보안을 담당하고 있는 인프라스트럭쳐 계층이라고 설명되어 있습니다. 
- Istio하면 따라다니는게 있는데 그건 바로 Envoy입니다.  istio sidecar인 **Envoy**가 CoreDNS 부하를 감소 시킬 수 있습니다.
- 짧게 설명해보자면, istio sidecar인 Envoy가 이미 Serive Discovery(FQDN)에 대한 servive-ip를 cache처럼 활용하는 것입니다.
- 자세하게 이해하고 싶다면, istio 공식 문서를 참고하시면 됩니다.

### 어떻게 설정할 수 있나요? 
현재 이기능은 기본적으로 istio를 설치를 진행하면 설정되지는 않습니다. 그래서 istio configmap를 추가 후 istiod pod를 재시작합니다.

![img_2.png](img_2.png)






