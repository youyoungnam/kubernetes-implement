## Volume
- 파드 내부의 각 컨테이너는 고유하게 분리된 파일시스템이 존재한다. 왜냐하면 파일시스템은 각 이미지에서 제공되기 때문이다. 만약에 컨테이너가 죽었다가 살아나게 된다면, 컨테이너가 이전에 실행했던 파일시스템의 어떤 파일들을 볼 수 없다.
- 그러나 우리는 실제로 물리 머신에서처럼 새로운 컨테이너가 다시 나와도 이전 데이터가 그대로 보존되고 싶을 때 가 있다. 그럴 때 사용하는게 볼륨이다.

### Volume LifeCycle
- 쿠버네티스 스토리지 볼륨은 파드의 일부분으로 정의 되며 파드와 동일한 라이프사이클을 가진다. 즉, 파드가 시작되면 볼륨이 실행되고 파드가 삭제되면 볼륨이 삭제된다는 것을 의미한다. 이 때문에 볼륨의 콘텐츠는 컨테이너를 다시 시작해도 지속된다. 즉, 컨테이너가 다시 시작되면 새로운 컨테이너는 이전 컨테이너가 볼륨에 기록한 모든 파일들을 볼 수 있다. 

## Volume Type
- emptyDir: 일시적인 데이터를 저장하는 데 사용되는 간단한 빈 디렉토리
- hostPath: 워커 노드의 파일시스템을 파드의 디렉토리로 마운트하는 데 사용
- gitRepo: 깃 레포지토리의 콘텐츠를 체크아웃해 초기화한 볼륨
- nfs: NFS 공유를 파드에 마운트
- gcePersistentDisk, awsElasticBlockStore, azureDisk: 클라우드 제공자의 전용 스토리지를 마운트
- cinder, cephfs, iscsi, flocker, glusterfs, qusterfs, quobyte, rbd, flexVloume, vsphere Volume, photonPersistentDisk, scaleIO: 다른 유형의 네트워크 스토리지 마운트
- configmap, secret, downwareAPI: 쿠버네티스 리소스 및 클러스터 정보 노출하는 데 사용하는 특별한 유형의 볼륨
- persistentVolumeClaim: 사전에 혹은 동적으로 프로비저닝된 퍼시스턴트 스토리지 사용 방법

단일 파드는 동시에 여러 유형의 여러 볼륨을 사용할 수 있다. 파드의 각 컨테이너는 볼륨을 마운트할 수 있고 하지 않을 수 있다. 

