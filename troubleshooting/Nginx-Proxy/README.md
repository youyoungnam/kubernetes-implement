## 클러스터 내에 Nginx Reverse Proxy 구축 경험
- 외부에서 사내 클러스터에 배포된 서비스를 접근해야 하는 로직이 생겼다. 외부에서 사내 클러스터 접근은 단순히 api를 통해서 매번 접근하는 건 어려운 일이다.(인증 서버를 개발하면 되지만, 빠르게 진행해야 하는 일이기 때문)
- 여러가지 방법을 생각했지만, 그중에 가장 괜찮은 방법은 클러스터 내에 하나의 vm을 만드는 것이다. 그래서 외부에서 클러스터 앞단에 nginx Proxy를 두고 클러스터 내에도 nginx proxy를 두는 것이다.
  
![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/18d64693-9dc3-4492-8cb5-5102370ac53c)


## nginx.conf 간단한 예제
```agsl
http { 
    server {
        listen       80;
        location /install {
 
        }
        
        location /delete {

        }
        
        location /update {

        }
    }
}
```



## cluster application nginx.conf 간단한 예제
```agsl
http { 
    server {
        listen       80;
        location /api/install {
             proxy_pass http:// service-name.namespace.svc.cluster.local/api/install ## 클러스터 내에 배포된 백엔드 파드
        }
        
        location /api/delete {
             proxy_pass http:// service-name.namespace.svc.cluster.local/api/delete ## 클러스터 내에 배포된 백엔드 파드
        }
        
        location /api/update {
             proxy_pass http:// service-name.namespace.svc.cluster.local/api/update ## 클러스터 내에 배포된 백엔드 파드
        }
    }
}
```

위 예제는 진짜 간단한 예제고 실제로 해보면 쉽게 구축할 수 있을 것이다. 사실 이방법은 프록시 방화벽을 잘 관리해야 한다는 단점이 있다.