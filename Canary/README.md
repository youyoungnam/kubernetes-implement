# Canary 
- 표준 쿠버네티스 방식으로 Canary 배포 방식 구현하기 
- 추후에는 Service Mesh - istio-방식으로 Canary 배포 방식 구현

## Standard Canary
- 하나의 서비스는 여러개의 파드를 가질 수 있다. 
- 쿠버네티스에서는 하나의 서비스에 다수의 Pod가 존재한다면, 파드를 찾는 방법은 Load Balancing 방식으로 찾아간다. 
- 만약 여기서 파드를 10%씩만 전달하게 하고싶다고 가정한다면, 파드를 10개를 만드는 방법도 괜찮은걸까? 심지어 여기에 단 특정 1%정도만 특정 파드에게 가도록 하고싶다면 어떤식으로 구현을 해야할까? 
- 만약 이것을 표준 쿠버네티스 방식을 구현하게 된다면 아래와 같이 구현할 수 있을거 같다. 

Example: 내가 이전에 Gitops ArgoCD 테스트한다고 이미지 만들어 놓은게 있는데 여기서 테스트해볼 수 있을거같다.
deployment와 service를 왜 이런식으로 만들었지 라고 생각할 수 있는데 빠르게 테스트하고 싶어서..

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-web-safe
spec:
  selector:
    matchLabels:
      app: python-service
  replicas: 2
  template:
    metadata:
      labels:
        app: python-service
        version: safe
    spec:
      containers:
        - name: python-service
          image: dbdudska113/flask_y:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-web-risk
spec:
  selector:
    matchLabels:
      app: python-service
  replicas: 1
  template:
    metadata:
      labels:
        app: python-service
        version: risk
    spec:
      containers:
        - name: vue-service
          image: dbdudska113/flask_y:v3
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: python-service
spec:
  selector:
    app: python-service
  ports:
    - name: http
      nodePort: 31532
      port: 8080
      targetPort: 5000
  type: NodePort
```
## 실험 

```
while true; do curl http://127.0.0.1:31532/name; echo; sleep 1; done
```

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/ba32e4a8-28c2-4868-b3b0-cb926052574e)



이미지를 보면 name이라는 api가 있는 version으로 파드를 호출하거나 api가 없는 version으로 호출하거나 한다.

---
## Istio-Canary
- Istio에서 Canary배포를 어떻게 구현할 수 있을까? 내가 알고 있는 방법이 100% 정답이라고 장담은 할 수 없지만 일단 도전을 해보는걸로 
- 회사내에서도 Istio Service Mesh를 사용중이기 때문에 연습이 필요하다.
- Istio를 활용해서 Canary배포를 연습하기 위해서 필요한건 virtual service와 DestinationsRule이다.

## Virtual Service
- 처음에 이스티오를 사용한다 했을 때 Virtual Service 단어는 너무나 어려워 보였다. 솔직히 어떤식으로 동작하는지도 전혀 몰랐고.. 쿠버네티스도 어려운데 Service Mesh 사용한다고 하니까 어려웠다. 

- 단어에 겁을 먹지말고, 내가 생각한 Virtual Service는 직관적으로 라우팅 규칙을 구성을 할 수 있는 것이라고 생각하면된다. 


- 이스티오를 사용한다면 Istio-pilot을 알고 있을 것이다 Istio-pilot은 Virtual Service 정의한 yaml를 가지고 있다고 생각하면 된다. 즉 파일럿은 즉시 실시간으로 Envoy Proxy를 구성하는 역할을 한다. 우리가 VS(Virtual Service)에 큰 변동사항이 있다고 가정하면 이러한 정보들이 파일럿으로 가고 파일럿이 사용자 지정 라우팅 규칙의 결과로서 변경되어야 할 모든 프록시에 전달한다. 

- 사실 모든 Canary를 배포하는건 envoy이다. 이스티오는 구성이 필요한 모든 프록시를 구성할 능력을 우리에게 주는 것이다. 우리는 사실 프록시에 대해 생각할 필요가 없다. 우리는 단지 높은 수준의 Virtual Service 개념에서 작업할 수 있다. 하지만 실제로 우리가 하는 일은 프록시를 구성하는 것이다. 




### VirtualService
```
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: a-set-of-routing-rules-virtual-service # Virtual Service 이름을 작성한다.
  namespace: default
spec:
  hosts:
    - python-service.default.svc.cluster.local #Service DNS 작성해야 한다.
  http:
    - route:
      - destination:
          host: python-service.default.svc.cluster.local 
          subset: safe 
        weight: 100
      - destination:
          host: python-service.default.svc.cluster.local
          subset: risky 
        weight: 0
```
### DestinationsRule
```
# 어떤 파드가 각각의 서브셋에 속할지 정하는 것
kind: DestinationsRule 
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: python-service-service-destination
  namespace: default
spec:
  host: python-service # Service
  trafficPolicy: ~
  subsets:
    - labels:  # Selector
        version: safe # 파드의 label "safe"
      name: safe
    - labels:
        version: risky # 파드의 label "risky"
      name: risky
```

---
