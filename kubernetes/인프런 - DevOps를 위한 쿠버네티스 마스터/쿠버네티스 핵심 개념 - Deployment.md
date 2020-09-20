# Deployment

### Deployment

- 애플리케이션을 다운 타임 없이 업데이트 가능하도록 도와주는 리소스
- 레플리카셋, 레플리케이션 컨트롤러 상위에 배포되는 리소스
    - yaml 템플릿도 이 둘과 유사

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-go
spec:
  replicas: 3
  selector:
    matchLabels:
      app: http-go
  template:
    metadata:
      labels:
        app: http-go
    spec:
      containers:
        - name: http-go
          image: gasbugs/http-go:v1
          ports:
            - containerPort: 8080
```

리소스 생성 후, 만들어진 리소스를 확인해보면, pod, replicaset 모두 만들어져 있음.

```bash
$ kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/http-go-5f5d8c9579-fnb8v   1/1     Running   0          46s
pod/http-go-5f5d8c9579-ghg5w   1/1     Running   0          46s
pod/http-go-5f5d8c9579-t9gh2   1/1     Running   0          46s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/http-go   3/3     3            3           46s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/http-go-5f5d8c9579   3         3         3       46s
```



### 롤링 업데이트와 롤백

- 새로운 포드를 실행시키고 작업이 완료되면 오래된 포드를 삭제
- Deployment 리소스 파일 내 `spec.strategy.type` 에서 `RollingUpdate` 값을 주면 됨.
    - 따로 입력하지 않아도 기본 값임.
    - `RollingUpdate` 말고도 `Recreate`과 같은 다른 배포 전략 선택도 가능함.
- 롤링 업데이트 전략 세부 설정
    - `spec.strategy.rollingUpdate` 내에 값을 주면 됨.
    - `maxSurge`: 최대로 추가 배포를 허용할 개수 설정 (기본값 25%)
        - 예를 들어, 떠있는 Pod가 4개인 경우 1개가 추가 배포되어 총 5개까지 동시 포드 운영
    - `maxUnavailabe`: 동작하지 않는 포드의 개수 설정
        - 예를 들어, 떠있는 Pod가 4개인 경우 1개가 설정되어, 총 3개는 운영해야 함.
- 업데이트를 실패하는 경우에는 기본적으로 600초 후에 업데이트를 중지한다.
    - `spec.processDeadlineSeconds: 600`

기본적으로 다음 명령어들이 있다.

```bash
# get deloyment <metadata.name> -o yaml 명령어로 자세히 확인할 수 있음
$ kubectl get deployment http-go -o yaml

# --record=true 옵션으로 히스토리 저장
$ kubectl create -f http-go-deployment-v1.yaml --record=true

# rollout status deploy <metadata.name> 명령어로 업데이트 상태 확인
$ kubectl rollout status deploy http-go

# rollout history deploy <metadata.name> 명령어로 업데이트 히스토리 조회
$ kubectl rollout history deploy http-go
```

업데이트는 다음과 같이 할 수 있다.

```bash
# v1 배포
$ kubectl create -f http-go-deployment-v1.yaml --record=true

# v2 배포
$ kubectl set image deployment http-go http-go=gasbugs/http-go:v2 --record=true
# (혹은 kubectl edit 을 사용해도 됨.) 

# 배포 히스토리 확인
$ kubectl rollout history deploy http-go
deployment.extensions/http-go
REVISION  CHANGE-CAUSE
1         kubectl create --filename=http-go-deployment-v1.yaml --record=true
2         kubectl set image deployment http-go http-go=gasbugs/http-go:v2 --record=true

# replicaset 확인 (v1 replicaset 의 D/C/R 이 모두 0/0/0 이 되어 있음)
$ kubectl get rs
NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/http-go-589b68f84b   3         3         3       3m41s
replicaset.apps/http-go-5f5d8c9579   0         0         0       9m54s
```

업데이트 취소는 다음과 같이할 수 있다.

```bash
# 업데이트 취소 (직전 리비전 버전으로 돌아가기) 
$ kubectl rollout undo deploy http-go
deployment.extensions/http-go rolled back

# 배포 히스토리 확인
$ kubectl rollout history deploy http-go
deployment.extensions/http-go
REVISION  CHANGE-CAUSE
2         kubectl set image deployment http-go http-go=gasbugs/http-go:v2 --record=true
3         kubectl create --filename=http-go-deployment-v1.yaml --record=true

# 특정 리비전 버전으로 돌아가기
$ kubectl rollout undo deploy http-go --to-revision=2
deployment.extensions/http-go rolled back
```

> **원하는 이미지 바탕의 리소스 yaml 파일 생성하는 꿀팁.**
>
> ```bash
> # kubectl create <resource> --image <image-name> --dry-run -o yaml 을 사용
> $ kubectl create deployment --image alpine:3.4 alpine-deploy --dry-run -o yaml > alpine.yaml
> ```
>
> ```yaml
> # alpine.yaml
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   creationTimestamp: null
>   labels:
>     app: alpine-deploy
>   name: alpine-deploy
> spec:
>   replicas: 1
>   selector:
>     matchLabels:
>       app: alpine-deploy
>   strategy: {}
>   template:
>     metadata:
>       creationTimestamp: null
>       labels:
>         app: alpine-deploy
>     spec:
>       containers:
>       - image: alpine:3.4
>         name: alpine
>         resources: {}
> status: {}
> ```

