## Kubernetes Pattern
- 앰버서더 컨테이너 패턴는 api 서버와 직접 통신하는 대신 메인 컨테이너 옆의 앰버서더 컨테이너에서 kubectl proxy를 실행하고 이를 통해 api 서버와 통신할 수 있는 Pattern이다. 
- api 서버와 직접 통신하는 대신 메인 컨테이너의 애플리케이션은 HTTPS 대신 HTTP를 실행하고 통신을 할 수 있다. 즉, 애플리케이션은 앰배서더와 연결하고 앰배서더는 서버에 대한 HTTPS 연결을 처리하는 것이다. 그리하여 보안을 투명하게 관리할 수 있다.


![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/c37a2eaf-f03b-40e1-92ef-5fb750e69a5d)

```agsl
apiVersion: v1
kind: Pod
metadata:
  name: appname
  labels:
    name: appname
spec:
  containers:
  - name: main
    image: tutum/curl
  - name: ambassador
    image: luksa/kubectl-proxy:1.6.2
```
현재 파드내 두개의 컨테이너가 존재하며 main container에 접근을 하려면 -c옵션을 사용해 접근해야한다. 

```agsl
kubectl exec -it appname -c main bash
```

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/1a336f29-7b3f-4cdd-a8ef-52603b34ee37)
