# Statefulset

### 스테이트풀셋 개념과 이해

- 스테이트풀셋으로 생성되는 포드는 영구 식별자를 가지고 상태를 유지할 수 있다.
- 스테이트풀셋을 사용하고자 하는 케이스
    - 안정적이고 고유한 네트워크 식별자가 필요한 경우
    - 안정적이고 지속적인 스토리지를 사용해야 하는 경우
    - 질서 정연한 포드의 배치와 확장을 원하는 경우
    - 포드의 자동 롤링업데이트를 사용하기 원하는 경우
- 포드 네트워크 ID를 유지하기 위해 헤드리스 서비스가 필요하다.
    - Service 리소스 선언에서 `spec.clusterIP: None` 으로 주면 됨.
- Pod가 삭제되도, PVC와 PV는 남아있음. (상태 유지)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
    
---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```



리소스를 생성하고 나면, 다음처럼 포드가 하나씩 순차적으로 뜨는 것을 볼 수 있음. (삭제할 때는 반대 순)

```bash
$ kubectl get all
NAME        READY   STATUS              RESTARTS   AGE
pod/web-0   1/1     Running             0          51s
pod/web-1   0/1     ContainerCreating   0          17s
```

PVC와 PV도 다음처럼 Pod 각각에 생긴 것을 볼 수 있음.

```bash
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
pvc-08ff4083-1499-4bb0-809a-186c94bd3b3b   1Gi        RWO            Delete           Bound    default/www-web-0   standard                6m30s
pvc-2956aa56-9d20-4903-86af-ba9ac3c465b2   1Gi        RWO            Delete           Bound    default/www-web-1   standard                5m57s

$ kubectl get pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE 
www-web-0   Bound    pvc-08ff4083-1499-4bb0-809a-186c94bd3b3b   1Gi        RWO            standard       6m38s
www-web-1   Bound    pvc-2956aa56-9d20-4903-86af-ba9ac3c465b2   1Gi        RWO            standard       6m4s
```

