Horizontal Pod Autoscaler는 Pod의 CPU 이용율을 관찰/측정하여 Deployment나 ReplicaSet에서 자동으로 Pods의 수를 Scale해줍니다.

Horizontal Pod Autoscaler를 사용하기 위해서는 Pod에 CPU 요청 및 제한이 정의되어 있어야 합니다

실습을 위해 사전에 container 이미지를 작성해 놓았습니다.

php로 작성되었으며, 요청을 받으면 1~1000000 모든 수의 제곱근을 구한 뒤 Hostname을 응답으로 주는 단순한 웹서버 입니다.

```php
<?php
  $x = 0.0001;
  for ($i = 0; $i &lt;= 1000000; $i++) {
    $x += sqrt($x);
  }
  echo gethostname();
  echo "\n";
?>
```

php 서버를 배포하는 yaml파일을 생성해 보겠습니다.

앞서 작성하였던 Deployment를 수정할 것이니 Editor 탭을 통해 파일을 수정하거나, `vi deployment.yaml`{{exec}} 를 통해 vi를 사용하셔도 됩니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment
  labels:
    app: httpd-deployment
spec:
  replicas: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: httpd-deployment
  template:
    metadata:
      labels:
        app: httpd-deployment
    spec:
      containers:
      - image: ethos93/myphp:3.0
        imagePullPolicy: Always
        name: httpd-deployment
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          requests:
            memory: "50Mi"
            cpu: "100m"
          limits:
            memory: "100Mi"
            cpu: "200m"
```

replicas 를 1로 변경해서 초기 1개의 Pod만 생성되게 한 뒤, image 를 ethos93/myphp:3.0 로 변경합니다.
그리고, Containers 에 resources를 추가하고, 초기 요청 memory와 cpu 사용량을 지정하고 제한 memory와 cpu 사용량을 지정해 줍니다.

Kubernetes의 CPU 1개는 클라우드 공급자용 vCPU/Core 1개 와 베어메탈 인텔 프로세서에서의 1개 하이퍼스레드 에 해당합니다. 또한 소수점도 허용됩니다.
0.1 CPU는 100m CPU와 동일합니다. CPU는 항상 절대 수량으로 요청되며, 상대적 수량은 아닙니다. 0.1은 단일 코어, 이중 코어 또는 48코어 시스템에서 동일한 양의 CPU 입니다.

Deployment를 수정하였다면, 변경사항을 반영합니다.

`kubectl apply -f deployment.yaml`{{exec}}

RollingUpdate를 통해 1개의 Pod 이미지가 교체된 뒤, 나머지 9개의 Pod은 자동으로 종료될 것입니다.

`kubectl get pods`{{exec}}

앞서 만들었던 curlpod를 통해 php 서버가 잘 동작하는지 확인해 봅니다.

`kubectl exec -it curlpod -- curl httpd-nodeport-service`{{exec}}

다음으로는 Horizontal Pod Autoscaler를 정의한 yaml파일을 생성해주어야합니다.

`touch hpa.yaml`{{exec}} 를 통해 다음을 선택하여 파일을 생성한 뒤 Editor 탭에서 아래 내용으로 hpa.yaml 파일을 완성시키거나, `vi hpa.yaml`{{exec}} 를 통해 vi를 사용하셔도 됩니다.

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: httpd-hpa
  namespace: default
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: httpd-deployment
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

최소 1개, 최대 10개 까지 Pod가 Scale되고, Pod 당 평균 CPU 사용량이 50%가 넘어가면 Scale Out될 수 있도록 지정 하였습니다.

이제 Horizontal Pod Autoscaler 로 적용해 줍니다.

`kubectl apply -f hpa.yaml`{{exec}}

적용 후에 `kubectl get hpa -o wide`{{exec}} 를 실행해 보면, 현재 적용되어 있는 hpa를 확인할 수 있습니다.

curlpod를 통해 php 서비스를 지속적으로 호출하여 부하를 발생시킵니다.

`while true; do kubectl exec -it curlpod -- curl httpd-nodeport-service; done`{{exec}}

Horizontal Pod Autoscaler 에서 maxReplicas의 수를 크게 지정하였더라도, Kubernetes Cluster가 처리할 수 있는 전체 cpu, memory Resource의 한계에 다다르면 더 이상 scale out이 되지는 않습니다.

다음과 같이 새로운 Tab을 열어 -w 옵션을 주고 get pods을 하면, 정보가 변경되는 내용이 지속적으로 출력되게 할 수 있습니다.

`kubectl get pods -w`{{exec}}

부하가 계속 주어질 때 Pod의 개수 변화를 확인 할 수 있습니다. (부하량이 충분하지 않으면, Pod이 빠르게 Scale Out 되지는 않습니다.)

while 문으로 부하를 주고 있는 terminal에서 <kbd>Ctrl</kbd>+<kbd>C</kbd> 로 부하를 멈추게 되면, 일정 시간이 지난 후에 다시 Pod의 개수가 변경되는 것을 확인할 수 있습니다.

`^C`{{exec}}
