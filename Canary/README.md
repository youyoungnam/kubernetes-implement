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
테스트 부분



이미지를 보면 name이라는 api가 있는 version으로 파드를 호출하거나 아니면 api가 없는 version으로 호출하거나 한다.
