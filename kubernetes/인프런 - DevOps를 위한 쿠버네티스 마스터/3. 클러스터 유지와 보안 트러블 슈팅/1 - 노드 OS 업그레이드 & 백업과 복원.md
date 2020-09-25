## 노드 OS 업그레이드 절차

- 노드를 갑자기 내리는 경우 다양한 문제가 발생할 수 있음.
- 포드는 노드가 내려가서 5분 이상 돌아오지 않으면, 그때 다른 노드에 포드를 복제
- 하나의 노드씩 다음 절차를 진행함
    - drain node
        - 포드를 모두 내리고.
        - cordon 명령 시행 (노드에 Pod이 들어올 수도, 나갈 수도 없게 함)
    - update node
        - 노드(OS, 컴퓨팅 리소스)를 업데이트 함
    - uncordon node
        - 노드에 Pod이 들어올 수 있도록



```bash
# 노드 이름 확인

$ kubectl get nodes
NAME                                       STATUS   ROLES    AGE   VERSION
gke-cluster-1-default-pool-ba1e1f3d-2to0   Ready    <none>   46m   v1.16.13-gke.401
gke-cluster-1-default-pool-ba1e1f3d-hrtu   Ready    <none>   20m   v1.16.13-gke.401
gke-cluster-1-default-pool-ba1e1f3d-uim8   Ready    <none>   41m   v1.16.13-gke.401
```

```bash
# 노드에 drain 명령
$ kubectl drain gke-cluster-1-default-pool-ba1e1f3d-uim8 --ignore-daemonsets

# 노드 상태 확인
$ kubectl get nodes
NAME                                       STATUS                     ROLES    AGE   VERSION
gke-cluster-1-default-pool-ba1e1f3d-2to0   Ready                      <none>   44m   v1.16.13-gke.401
gke-cluster-1-default-pool-ba1e1f3d-hrtu   Ready                      <none>   18m   v1.16.13-gke.401
gke-cluster-1-default-pool-ba1e1f3d-uim8   Ready,SchedulingDisabled   <none>   40m   v1.16.13-gke.401  # SchedulingDisabled 되어 있음!
```

```bash
# 노드의 cordon 상태 풀기
$ kubectl uncordon gke-cluster-1-default-pool-ba1e1f3d-uim8

# 노드 상태 확인
$ kubectl get nodes
NAME                                       STATUS   ROLES    AGE   VERSION
gke-cluster-1-default-pool-ba1e1f3d-2to0   Ready    <none>   46m   v1.16.13-gke.401
gke-cluster-1-default-pool-ba1e1f3d-hrtu   Ready    <none>   20m   v1.16.13-gke.401
gke-cluster-1-default-pool-ba1e1f3d-uim8   Ready    <none>   41m   v1.16.13-gke.401
```



## 백업과 복원 방법

- 포드의 정보 파일 yaml
    - 다음과 같이 `yaml` 로 백업 가능.
    - `kubectl get all -A -o yaml > all-deploy-services.yaml`
    - 다음과 같이 복원 가능.
    - `kubectl create -f all-deploy-services.yaml`
- Etcd 데이터베이스
    - Configmap, Secret, PVC 등의 정보는 etcd 에서 확인해야 함.
    - 별도의 etcd 데이터 베이스 백업 방법을 따름.
- PV
    - 일반적인 방법으로 백업

(실습 생략)