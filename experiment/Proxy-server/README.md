# 클러스터 앞단에 Proxy Server를 둔 이유
- 개발환경 클러스터 앞단에 Proxy Server를 둔 이유는 개발환경에 ssl를 적용하면 보통 443 포트만 외부로 노출이된다. 그렇다면 클러스터내에 설치된 개발 tool들은 접근을 할 수 없다. 
- 아래 이미지 처럼 443 포트 이외엔 접근할 수 가 없다. 그리고 SSL이 적용 했는데 다른 포트를 노출시킨다는건 이상하다.
![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/2585b6b6-db62-4ebd-859c-78b2b2ab54df)


  
## 해결방법 
- 그렇다면 어떤식으로 해결할 수 있을까? Proxy Server를 따로 운영해서 개발자들에게 접근할 수 있게 해주는 방법밖에 없다. 그러면 Proxy Server 생성 위치도 중요하다.Proxy Server 위치는 클러스터 내에 있는 노드와 같은 VPC내에 생성되어야 한다. 그래야지 노드내에 배포된 개발 tool들을 접근할 수 있다.
  ![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/50975425-3cd1-4988-ad3a-d39566eaa3cc)
- Proxy 구성은 nginx도 괜찮고 squid proxy도 괜찮다.


## 시도한 방법
- 맨 처음 시도한 방법은 proxy server를 클러스터 노드와 같은 vpc에 생성을 하고 proxy server에서 노드들과 연결할 수 있는지 테스트를 시도했을 때 노드들과 연결은 잘 되는것을 확인했다. 그렇다면 이제 노드 내에 배포된 파드들에게 접근이 가능한지 테스트를 했는데 접근이 잘 되었다. 
- 모두다 잘 알고 있겠지만 파드는 일시적이다. 파드가 죽으면 파드 ip는 변경된다. 그렇다면 파드를 프록시를 연결한다면 파드가 죽었다가 살아나거나 배포를 다시하게 된다면 프록시 설정을 계속 변경해줘야 한다는 점이 생긴다. 그러면 서비스를 통해 접근을 해야한다. 
- 여기서 놓친게 있었다. service ip는 가상의 ip라는 점이다. veth0 즉 클러스터 내 리소스들과 통신할 수 있는 ip지 외부에서 접근할 수 있는 ip가 아니라는 소리다.
- 그렇다면 어떤식으로 해야할까? 그 다음 시도한 방법이 service type를 NodePort로 노출시킨다면 노드 IP로 접근할 수 있겠네? 했으나 연결이 되지않았다 그렇다면 다른 방식으로 접근해야 하는데 쿠버네티스 파드 옵션중에 hostPort라는 옵션이 존재한다. hostPort 옵션을 사용하게 된다면 특정 노드에 배포된 파드가 node의 network namespace를 공유한다는 것이다. 
- 그렇다면 서비스를 건들지 않아도 파드의 호스트 포트로 노출시킨다면 node-ip:port를 통해 접근할 수 있다. 

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/9561261c-8c69-463e-9a70-f867ede84e97)
