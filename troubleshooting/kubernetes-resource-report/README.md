## Kubernetes Resource Report
- kubernetes cluster 내에 application에 대한 resource requests and usage를 report로 보여주는 tool이다.
- 해당 프로젝트의 목표는 kubernetes 리소스 요청을 최적화하는게 목표다. 
- 여기서 Slack이라는 단어가 사용되는데 slack은 application resource requests 과 실제 resource 차이이다. 
- 예를들어 특정 Application requests memory가 2GiB를 요청중이고 실제 운영할 때 사용중인 memory가 200MiB만 사용하고 있으면
여유분이 1.8GiB가 발생하게 된다. 그러면 해당 메모리 용량이 사용하고 있지 않지만 비용은 나가고 있다는 것 


## install 이슈 
- 해당 application를 클러스터 내에 배포하고 port-foward를 통해 접속하게 되면 아래 처럼 Error가 나타나는걸 볼 수 있다. 
```agsl
Not able to setup kube-resource-report in my k8s env and getting 403 Client Error Forbidden for url
```
찾아보니까 해당 application을 배포할 때 rbac역시 같이 배포하게 되는데 serviceAccount 배포되는 곳이 default로 설정되어 있어서 특정 네임스페이스에 배포할 때 권한이 없어 정보를 못가져오는 이슈이다.
즉, serviceAccount를 올바른 곳에 배포하지 않아서 생기는 이슈였다. 나는 monitoring namespace에 배포해야하는데 ServiceAccount혼자 default에 배포되어 문제가 생긴것 그냥 rbac.yaml에 namespace만 
설치가 원하는 곳으로 변경만 해주면 쉽게 해결할 수 있다. 
