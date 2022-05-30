컨테이너 CPU 및 메모리 사용량과 같은 리소스 사용량 메트릭은 쿠버네티스의 메트릭 API를 통해 사용할 수 있습니다. 이 메트릭은 kubectl top 명령 사용과 같이 사용자가 직접적으로 액세스하거나, Horizontal Pod Autoscaler 같은 클러스터의 컨트롤러에서 결정을 내릴 때 사용될 수 있습니다.

이 메트릭을 사용하기 위해서는, 메트릭 서버를 사전에 Kubernetes Cluster에 배포를 하여야만 합니다. (Killercoda 도 사전에 배포되어 있지 않음)

Kubernetes가 정상적으로 구동이 되었는지 확인을 한 뒤에 메트릭 서버를 배포합니다.

`kubectl get nodes`{{execute}} 를 실행하여, 두개의 node가 모두 Ready 인 것을 확인한 후 진행합니다.

메트릭을 사용하기 위한 Manifest는 /root/metrics-server/components.yaml 에 구성되어 있으니, 이를 먼저 배포 합니다.

`kubectl apply -f /root/metrics-server/components.yaml`{{execute}} 를 실행하면, 메트릭 서버가 배포됩니다.

배포된 이후에 일정시간이 지나면 메트릭 정보를 수집하여 kubectl top 명령을 통해 확인 할 수 있습니다.

Kubernetes Deployment 전략을 지정하고 Rollback을 실습하기 위해 앞서 생성했던 것과 동일한 Deployment를 생성해 보겠습니다.

`touch deployment.yaml`{{execute}} 를 통해 다음을 선택하여 파일을 생성한 뒤 Editor 탭에서 아래 내용으로 deployment.yaml 파일을 완성시키거나, `vi deployment.yaml`{{execute}} 를 통해 vi를 사용하셔도 됩니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deployment
  labels:
    app: httpd-deployment
spec:
  replicas: 10
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
      - image: ethos93/go-httpd:v1
        imagePullPolicy: Always
        name: httpd-deployment
        ports:
        - containerPort: 80
          protocol: TCP
```

이제 `kubectl apply -f deployment.yaml`{{execute}} 명령을 통해 Deployment 을 생성합니다.

다음으로, 앞서 작성한 것과 동일한 NodePort Type의 서비스도 생성해 보겠습니다.

`touch nodeport_svc.yaml`{{execute}} 를 통해 다음을 선택하여 파일을 생성한 뒤 Editor 탭에서 아래 내용으로 nodeport_svc.yaml 파일을 완성시키거나, `vi nodeport_svc.yaml`{{execute}} 를 통해 vi를 사용하셔도 됩니다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-nodeport-service
spec:
  selector:
    app: httpd-deployment
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

`kubectl apply -f nodeport_svc.yaml`{{execute}}

아래 명령을 통해 debugging용 pod을 생성합니다.

curl을 포함하고 있는 아주 작은 container image 입니다.

`kubectl run curlpod --image=radial/busyboxplus:curl --command -- /bin/sh -c "while true; do echo hi; sleep 10; done"`{{execute}}

curlpod 라는 pod 이 생성되었으니, 이제 curlpod 에서 서버스 이름으로 http 서버를 호출해 보겠습니다.

`kubectl exec -it curlpod -- curl httpd-nodeport-service`{{execute}} 으로 호출해 보면, curlpod 안에서 curl 명령이 실행됩니다.

