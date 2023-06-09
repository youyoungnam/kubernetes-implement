# service


### 서비스 동작방식
- 우리는 우리가 직접 컨테이너를 동작시키거나 생성하거나 하지않는다. 회사에서 사용하는 것을 생각해보면 우리가 직접 컨테이너를 동작시키지 않는다. 즉, 쿠버네티스가 마스터 서버에서 api와 컨트롤러와 소통을 하고 워커노드와 통신 후 적당한 노드에 배치한 다음에 노드에 존재하는 kube-proxy, kubelet, docker가 이미지를 불러오고 실행한다. 
- 만약에 우리가 kubectl run을 한다고 가정했을 때 레플리케이션 컨트롤러를 생성하고 러플리케이션 컨트롤러가 실제 파드를 생성한다. 클러스터 외부에서 파드에 접근케 하기 위해 쿠버네티스에게 레플리케이션 컨트롤러에 의해 관리 되는 모든 파드를 단일 서비스로 노출하도록 명령한다. 


client(포트 8080) ---------> 서비스: 서비스네임--> youngnam-http 내부ip: 10.x.x.x 외부ip: 104.x.x.x -----> 포트 8080 ----> 파드: application-andy(container) ----> 레플리케이션컨트롤러: 1

### 서비스는 왜 필요한 것인가? 
- 서비스가 왜 필요한지에 대해서 알려면 파드에 대해서 알아야한다. 파드는 일시적이다. 즉, 죽으면 ip가 변한다. 그리고 언제 어디서든지 갑자기 사라질 수 도 있다. 그래서 레플리케이션은 desired state를 만족하기 위해 다시 생성한다. 이렇게 만들어진 새로운 파드는 새로운 ip를 가지고 있다. 이것이 진짜 서비스가 필요한 이유다. 항상 변경되는 ip 주소 문제와 여러 개의 파드를 단일 ip와 포트의 쌍으로 노출 시키는 문제를 해결한다. 
- 서비스가 생성되면 정적 ip를 할당받는다. 서비스가 존속하는 동안은 변경되지 않는다. 파드에 직접 연결하는 대신 클라이언트는 서비스의 ip주소만으로 연결할 수 있다. 
- 서비스는 서비스가 존재하는 동안 절대 바뀌지 않는 ip와 포트가 존재한다. 클라이언트는 해당 ip와 포트로 접속한 다음 해당 서비스를 지원하는 파드 중 하나로 연결된다. 

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/62f3790d-04ec-4b76-9f88-4a2b34046f89)

위 그림을 보면 서비스를 지원하는 파드가 한 개 혹은 그 이상일 수 도 있다. 서비스 연결은 뒷단의 모든 파드로 로드밸런싱된다. 그러나 정확히 어떤 파드가 서비스의 일부분인지 아닌지 정확히 어떻게 정의할까???
레이블 셀렉터 Label Selector를 기억할 수 있다. 그리고 레이블 셀렉터는 레플리케이션 컨트롤러와 기타 파드 컨트롤러에서 레이블 셀렉터를 사용해 동일한 세트에 속하는 파드를 지정하는 방법을 기억할 것이다. 


### 특정 파드내에서 다른서비스 curl 날리는법
```agsl
kubectl exec pod-name -- curl -s http://service-ip
```