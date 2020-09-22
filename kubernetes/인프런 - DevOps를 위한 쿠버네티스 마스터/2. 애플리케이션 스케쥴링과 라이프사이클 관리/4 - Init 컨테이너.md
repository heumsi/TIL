# Init 컨테이너

- 포드 컨테이너 실행 전에 초기화 역할을 하는 컨테이너
- 완전히 초기화가 진행된 다음에야 주 컨테이너를 실행
- Init 컨테이너가 실패하면, 성공할 때까지 포드를 반복해서 재시작
    -  `restartPolicy: Never` 를 하면 재시작을 하지 않음.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    
  # 여기서부터가 init 컨테이너!
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```