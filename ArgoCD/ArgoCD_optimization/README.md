# ArgoCD 운영 최적화


최근에 운영중이던 클러스터가 자주 죽는 이슈가 발생했습니다. 이유를 확인해보니 ArgoCD 때문인것을 확인했습니다. 최적화를 진행하기전 초기에 ArgoCD를 배포했을때 구성으로 운영해왔습니다. 

문제가 발생하기 시작했던 시점은, 점차늘어나는 클러스터, 점점 늘어나는 Application 수가 문제 였습니다. 단일 ArgoCD에는 10개이상의 클러스터가 연결되어 있는 상태였고, Application 갯수는 300+이였습니다.


## 발견된 문제점
- 단일 Application-controller cpu, memory 스파크
- CICD 진행했을 때 Argocd repo server cpu 스파크 


## 적용해본 최적화 방식
- pull 방식을 push 방식으로 변경
- argocd-application HA 
- Argocd repo server option 최적화 




### Application Controller 최적화 
- HA 구성
  - ArgoCD가 관리하는 클러스터가 점차 많아질 수 록 ArgoCD의 Application Controller는 더 많은 cpu와 memory가 소비될 수 있습니다.특이점이 왔을 때 Replicas를 늘려 관리하는 클러스터를 나눠서 운영할 수 있게 해주는게 좋습니다.
  - 단순히 replicas만 늘린다고 해서 해결되지는 않고,  Kubernetes 클러스터의 서브셋을 담당하도록 구성해야 합니다. 해당값 조정은 ARGOCD_CONTROLLER_REPLICAS option을 조절하면 됩니다. 



### pull 방식에서 push 방식으로 변경 
- ArgoCD는 default로 pull 방식을 통해서 매 3분마다 application manifest를 체크해서 변경사항을 체크합니다. 
- 단일 Application update될 때마다 전체 레포에 대한 캐시가 무효화가 됩니다. 
- 변경된 Application만 Webhook 트리거를 통해 argocd에게 전달을 하고 동기화를 진행할 수 있도록 했습니다. 
```shell
# bitbucket push webhook
 k get secret -n argo argocd-secret 
  # bitbucket webhook Header에 있는 X-Hook-UUID값으로 넣어주기 
  webhook.bitbucket.uuid: bitbucket webhook X-Hook-UUID 
  

# argocd application annotation
apiVersion:  argoproj.io/v1alpha1 
kind:  application
metadata: 
  name:  example-guestbook 
  네임스페이스:  argo
  annotations: 
    # 'guestbook' 디렉토리로 경로
    # 동기화될 application 위치를 작성해야합니다. 
    argocd.argoproj.io/manifest-generate-paths:  . 
spec: 
  source: 
    repoURL:  https://argocd-repo.git 
    targetRevision:  HEAD 
    path:  guestbook 
```
참고자료
- https://argo-cd.readthedocs.io/en/stable/operator-manual/high_availability/


## Argocd repo server option 최적화
```shell
 reposerver.enable.git.submodule: "false"
 reposerver.kubectl.parallelism.limit: "7"
 reposerver.parallelism.limit: "1" 
 reposerver.repo.cache.expiration: 6h
```
- 첫 번쨰 옵션은 서브모듈이 필요 없을 경우 false로 합니다. 
- 두 번째 옵션은 ArgoCD가 kubectl을 실행할 때 동시에 실행할 수 있는 최대 병렬 프로세스 개수를 제한합니다.
- Git 리포지토리 클론, Helm/Kustomize 템플릿 렌더링 등의 작업을 동시에 몇 개까지 실행할지 결정.
  - 1로 설정하면 한 번에 하나의 리포지토리만 처리 가능 → 메모리 사용량 최소화. 하지만 여러 개의 애플리케이션을 동시에 배포할 경우 성능이 저하될 수 있음.
- 세 번째 옵션은 Git 리포지토리 캐시(템플릿, Helm 차트 등)의 유효 기간을 설정. 캐시가 만료되면 새롭게 Git 리포지토리를 다시 클론하고 데이터를 갱신함.

💡 현재 설정 (reposerver.parallelism.limit: 1, reposerver.kubectl.parallelism.limit: 7)의 의미

만약 두 개의 애플리케이션(A, B)이 동기화될 때

Git에서 A, B 둘 다 데이터를 가져와야 함 → reposerver.parallelism.limit 영향
A, B의 Kubernetes 리소스를 kubectl로 적용해야 합니다 → reposerver.kubectl.parallelism.limit 영향을 줄 수 있습니다.
한 번에 **한 개의 Git 리포지토리(A 먼저, 끝나면 B)**만 처리합니다. 하지만, kubectl은 한 번에 7개까지 명령을 실행할 수 있으며, 따라서 배포 자체는 병렬로 빠르게 진행될 수 있습니다.


위 옵션은 운영환경에 맞게 설정을 하셔야합니다. 

참고
- https://argo-cd.readthedocs.io/en/release-2.2/operator-manual/server-commands/argocd-repo-server/
- https://argo-cd.readthedocs.io/en/stable/operator-manual/argocd-cmd-params-cm-yaml/
- https://argo-cd.readthedocs.io/en/stable/operator-manual/high_availability/