## istio operator를 통해 istio 설치하기

- 사내에서도 service mesh로 istio로 사용중인데 아무리 개발환경이 존재해도 istio를 막 테스트하기엔 쉽지않다. 그래서 로컬에 istio를 설치해서 여러가지를 테스트해보려고 생겼던 이슈 였다.

## istio operator를 통해서 설치하기 
- 개발환경 구성할 때도 istio operator를 통해 설치하긴 했지만, 옵션을 조금 더 보고싶었고 설치중에 테스트해보고 싶은게 있었다. 
- 사실 istio label를 수정하고 싶었는데 아무리 istio operator를 수정하고 설치해도 내가 원하는 label로 설치되지 않았다. 
- 그래서 공식문서에 나와있는 yaml를 참고해서 설치하지 않고 istio 내에 있는 profiles 폴더내에 있는 yaml를 통해서 label를 수정하고 설치하니 내가 원하는 label로 설치되었다.


```agsl
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  meshConfig:
    accessLogFile: /dev/stdout
    extensionProviders:
    - name: otel
      envoyOtelAls:
        service: opentelemetry-collector.istio-system.svc.cluster.local
        port: 4317
    - name: skywalking
      skywalking:
        service: tracing.istio-system.svc.cluster.local
        port: 11800
    - name: otel-tracing
      opentelemetry:
        port: 4317
        service: opentelemetry-collector.otel-collector.svc.cluster.local
  components:
    egressGateways:
    - name: istio-egressgateway
      enabled: true
      k8s:
        resources:
          requests:
            cpu: 10m
            memory: 40Mi

    ingressGateways:
    - name: istio-ingressgateway
      label:
        istio: gateway-local # istio svc istio label 수정하기 
      enabled: true
      k8s:
        resources:
          requests:
            cpu: 10m
            memory: 40Mi
        service:
          ports:
            - port: 15021
              targetPort: 15021
              name: status-port
            - port: 80
              targetPort: 8080
              name: http2
            - port: 443
              targetPort: 8443
              name: https
            - port: 31400
              targetPort: 31400
              name: tcp
            - port: 15443
              targetPort: 15443
              name: tls

    pilot:
      k8s:
        env:
          - name: PILOT_TRACE_SAMPLING
            value: "100"
        resources:
          requests:
            cpu: 10m
            memory: 100Mi

  values:
    global:
      proxy:
        resources:
          requests:
            cpu: 10m
            memory: 40Mi

    pilot:
      autoscaleEnabled: false

    gateways:
      istio-egressgateway:
        autoscaleEnabled: false
      istio-ingressgateway:
        autoscaleEnabled: false
```