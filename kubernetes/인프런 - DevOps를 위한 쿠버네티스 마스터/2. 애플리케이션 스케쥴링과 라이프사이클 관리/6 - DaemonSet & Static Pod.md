# DaemonSet & Static Pod

## DaemonSet

- 레플리케이션 컨트롤러와 레플리카셋은 무작위 노드에 포드를 생성함.
- 데몬셋은 각 하나의 노드에 하나의 포드만을 구성
- kube-proxy가 데몬셋으로 만든 쿠버네티스에서 기본적으로 활동 중인 포드

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: http-go
spec:
  selector:
    matchLabels:
      app: http-go
  template:
    metadata:
      labels:
        name: http-go
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: http-go
        image: gasbugs/http-go
```



## Static Pod

- 노드에서 kubelet이 직접 실행하는 포드
- 노드의 `/etc/kubernetes/manifests/` 경로에 리소스 `.yaml` 을 넣어주면 됨.
- 실행을 위한 별동의 명령어는 필요하지 않음.
- `kubectl delete` 해도 삭제되지 않음.
- 이름이 `{pod-name}-master` 와 같은 포드가 마스터 노드에서 띄운 Static Pod 임.

