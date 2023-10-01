## coreDNS란 무엇일까?
- coredns는 쿠버네티스 클러스터 dns서버 역할을 담당한다. k8s 애드온으로 쿠버네티스 실행시 pod로 실행된다. 
- 파드, 서비스 도메인을 관리하고 외부 도메인 서버 연결이 필요한 경우 통로역할을 수행
- 파드 내부에 접속해서 etc/resolv.conf파일을 확인하면 nameserver에 kube-dns로 바라보고 있는걸 확인할 수 있다.

kubedns cluster ip

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/34e54dbf-b124-4129-bf48-6512e37174c4)


파드 내부 nameserver

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/652b5fa1-8298-434c-a000-0f0302646f65)

## 이게 무슨말일까? 
- 파드는 dns 요청할 때 kube-dns service를 먼저 찾는다는 것이다. 
- 즉, coredns는 파드 or 서비스를 도메인으로 접근할 수 있다는 것이다. 
- 파드나 서비스는 어떠한 변경이 일어나면 cluster ip가 변경되는데 이러한 문제를 coreDNS로 해결가능하다. 
- 파드 도메인: pod-ip.namespace.pod.cluster.local
- 서비스 도메인: service-name.namespace.svc.cluster.local

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/849f5411-167d-484e-ac62-c65d745a262b)
A라는 파드가 B라는 파드를 찾기위해서 이러한 과정이 있을것이다.
- A라는 파드는 먼저 kube-dns service에 요청을 할것이고 kube-dns service는 kube-dns파드에 접근해서 B라는 서비스 dns가 있는지 물어볼 것이고 
- 존재한다면 cordns pod는 다시 A라는 파드에게 전달이 될것이고 A라는 파드는 B라는 서비스를 요청해서 B라는 파드에 접근할 수 있는 것이다. 

