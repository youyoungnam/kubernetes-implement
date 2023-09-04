## Istio 모니터링을 위한 Kiali 설치하기 
![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/b47876e9-e18b-4da5-b4f5-abe3929a071e)

- istio를 설치하게 되면 istio 폴더 내에 addon에 kiali가 있는것을 확인할 수 있다. addon에 있는 Kiali를 설치하게 되면 yaml에 설정되어 있는대로 설치가 진행된다.
- 아마도 istio-system 네임스페이스에 설치가 될것이고 promethues역시 설치해야하기 때문에 prometheus도 마찬가지로 istio-system에 설치가 될것이다. 
- 근데 나는 이것을 이용햇 우리가 구축한 monitoring이라는 네임스페이스에 따로 설치하고 싶은데 그대로 사용하면 안될거 같아서 yaml내부를 보았다. 
- 일단 시도했던 방법은 istio-system네임스페이스로 설정되어 있는 부분들을 monitoring으로 변경했는데 하나씩 자세히 봐야한다. 배포하더라도 전혀 동작하지 않는다. 

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/9bc34221-eeb0-4c1b-b6d8-ed7dad768742)

첫 번째 이슈는 이부분에서 잘못 설정을 해서 배포하고도 Kiali가 제대로 동작하지 않았다. 찾아보니까 Kiali는 istio resource와 관련된 루트 네임스페이스를 사용한다. 그래서 kiali를 제대로 동작하게 하려면 istio의 root namespace를 구성한곳에
지정해야 한다는 것이다. 즉 istio를 설치할 때 root namespace를 지정하는 부분이 있는데 그 네임스페이스를 작성해야한다는 것 

![image](https://github.com/youyoungnam/kubernetes-implement/assets/60678531/10db0cb2-8cf7-464f-8ee0-eb0c40798433)

두 번째 이슈는 더이상 kiali에서 istio관련 에러는 나오지가 않았으나, prometheus 에러는 여전히 남아있었다. kiali를 설치할 때 prometheus를 따로 지정하지않으면 prometheus 서비스 디스커버리 주소를 istio-system으로 잡는다. 
그래서 문서를 찾아보니까 prometheus.istio-system:9090으로 되어있는 것을 확인했다. 이부분을 우리 모니터링 네임스페이스에 배포되어 있는 prometheus로 변경해야 한다. 그래야 metrices를 가져올 수 있다. 

