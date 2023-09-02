## k8s 클러스터 내에 redis sentinel 구성하기
- 사내 클러스터내에 배포되어 있는 애플리케이션이 사용중인 Redis가 있는데 이 Redis는 Master & Slave로 구성되어 있었다. 사실 Master & Slave도 괜찮은 방법이지만, 단점은 만약에 마스터가 죽어버린다면 Slave는 읽기밖에 못하기 때문에 서비스가 중단될 수 있다. 
- 이 문제점을 해결하기 위한 방식이 Sentinel방식이다. Sentinel 서버는 마스터와 슬레이브를 계속 모니터링하다가 마스터가 죽을 경우 슬레이브를 마스터로 변경해서 데이터 유실이 발생하지 않도록 Fail-over하는 역할이다.


## 이슈
- 우리는 쿠버네티스 환경을 운영하면서 kustomize를 사용하고 있다. 개발환경과 운영환경 두 곳을 클러스터로 운영을 하고 있기 때문에 Redis를 배포할 때 역시 kustomize를 사용해서 배포를 진행하려고 했다.
- Redis Sentinel를 처음 구축하다보니 개발환경에 먼저 테스트를 진행했다. 문서와 구글을 찾아보면서 Sentinel를 구축했으나 Sentinel이 배포될 때마다 계속 죽는 상태를 봤다. 
- Sentinel 에러를 봤을 때 sentinel를 배포할 때 스크립트를 실행시키는데 이 스크립트에서 마스터를 지정하는 부분이 있는데 이부분에서 에러가 났다. 
- 보니까 Redis master 1대 Slave 2대 총 3대가 먼저 배포가 되는것을 확인 후 sentinel에서 마스터를 찾는 방식으로 진행해야 한다. 
- 즉 나는 kustomize를 구성한 후 한번에 배포를 해버리니까 Sentinel에서 redis가 배포를 완료하기 전부터 마스터를 찾으니까 없는 마스터를 찾아서 죽는것이 였다. 
- Redis Master & Slave & Sentinel를 구축할 때 순서를 통해 배포를 진행하자
- 순서
  - redis.config.yaml
  - redis.statefulset.yaml
  - redis.sentinel.yaml
- 이런식으로 순서를 정해서 배포해야만 sentinel를 구성할 수 있다.

