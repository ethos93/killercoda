Kubernetes에 Application을 배포하고, 외부에서 접속 할 수 있도록 설정을 하는 실습을 해 보겠습니다.

현재 kubernetes 환경의 상태를 확인하기 위해서는,

`kubectl cluster-info`{{exec}}

`kubectl get nodes`{{exec}}

`kubectl get pods -A`{{exec}}

를 각각 실행시켜 보시면 됩니다.

Killercoda의 Kubernetes Cluster 환경은, controlplane 하나의 host로 구성되어 있으며, 시나리오에 따라 controlplane 과 node01 두개로 구성되기도 합니다.

kubernetes의 최소 단위는 POD 이며, POD의 Replica 개수를 지정하는 것이 ReplicaSet, 그리고 ReplicaSet의 배포 및 롤백과 같이 History를 포함하는 것이 Deployment 입니다.
