## Prometheus + Grafana를 운영하기 시작한 이유
- 우리 회사의 모든 서비스는 쿠버네티스위에서 동작하고 있다. 모든 서비스를 쿠버네티스 위에서 운영하다보니, 다른팀에서 만든 애플리케이션 모니터링할 수 있는 방법이 따로 없었다.
- 다른 팀에서 빠르게 모니터링(서버 로그를 볼 수 있는정도) 할 수 있는 방법이 팀마다 클러스터에 접근할 수 있게 하고 [workload-Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity?hl=ko)를 부여 후 제품이 배포되어 있는 네임스페이스에만 권한을 주는 것이다. 즉 로그만 볼 수 있는게 만들었다. 
  - 이부분에도 단점이 존재했다. 사내 모든 팀들에게 클러스터를 접근하게 만든것도 문제고 모든 팀에게 workload identity를 만들어야하는 이슈가 존재했다. 
  - 사실, 팀마다 Service Account IAM은 클러스터 뷰어권한밖에 없지만, 보안상 문제가 있을거라 생각했다. 


## 모니터링 도구 Prometheus + Grafana
- prometheus를 따로 설치하고 쿠버네티스를 연결하는것도 하나의 방법이지만, 모든 사내 애플리케이션이 쿠버네티스 위에서 동작하고 있고 설치도 빠르게 할 수 있기 때문에 쿠버네티스 클러스터 위에 배포했다.

## Prometheus 설치 및 이슈
- 처음에 helm Chart를 통해서 prometheus를 설치해보면 알겠지만, 기본적인 것들만 수집을 하고있다는것을 볼 수 있다. 우리는 service mash를 istio를 선택하고 사용중인데, istio에 대한 메트릭은 전혀 수집을 하고 있지 않았다.
- kafka 역시 메트릭을 수집하려면 exporter 배포하고 prometheus에 연결해야 하는 이슈가 존재했다. 


## 해결법
- [chart](https://github.com/youyoungnam/kubernetes-implement/tree/main/troubleshooting/prometheus/chart.yaml)
- [kafka-exporter](https://github.com/youyoungnam/kubernetes-implement/tree/main/troubleshooting/prometheus/kafka-exporter.sh)