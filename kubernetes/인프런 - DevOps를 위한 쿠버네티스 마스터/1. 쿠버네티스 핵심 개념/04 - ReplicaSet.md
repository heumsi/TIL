# ReplicaSet

### 레플리케이션 컨트롤러

- 포드가 항상 실행되도록 유지하는 쿠버네티스 리소스
- 노드가 클러스터에서 사라지는 경우, 해당 포드를 감지하고 대체 포드 생성
- 실행 중인 포드의 목록을 지속적으로 모니터링하고, 실제 포드 수가 원하는 수와 항상 일치하는지 확인
- Pod이 장애가 발생한 경우, 5분 뒤 해당 Pod을 내리고 새로운 Pod 생성
    - 5분은 기본 설정 값임 (수정가능).
    - 일시적 장애일 수 있기 때문에 5분이라는 유예 시간을 주는 것. (5분이 아니라 즉각즉각 생성하면 Pod 과다 생성할 수 있음. 이를 방지)
- 레플리케이션 컨트롤러의 세가지 요소
    - 레이블 셀렉터: 관리해야하는 포드 범위
    - 복제본 수: 실행해야하는 복제본 수
    - 포드 템플릿: 포드의 모양 설명

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: http-go
spec:
  replicas: 3  # 복제본 수
  selector:  # 레이블 셀렉터
    app: http-go
  template:  # 포드 템플릿 (apiVersion과 kind만 없음)
    metadata:
      name: http-go
      labels:
        app: http-go
    spec:
      containers:
        - name: http-go
          image: heumsi/go-tutorial-app
          ports:
            - containerPort: 8080
```

그 외 명령어

```bash
# 복제본 수 변경하기 (scale)
$ kubectl scale rc http-go --replicas=5

# yaml 형태의 임시파일에 직접 수정 (edit)
$ kubectl edit rc http-go
```



### 레플리카셋

- 쿠버네티스 1.9 버전부터 Deployment, Daemonset, ReplicaSet, StatefulSet 이 정식버전으로 업데이트 됨. (1.8에서는 베타버전)
- 레플리카셋과 레플리케이션 컨트롤러는 거의 동일하게 동작
    - 다만, 레플리카 셋이 더 풍부한 표현식 포드 셀렉터 사용 가능
- 추후 레플리카셋이 레플리케이션 컨트롤러를 완전히 대체할 예정

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-nginx
spec:
  replicas: 3
  selector:
    matchLabels:  # matchLabel 등 다양한 방식으로 Pod 선택 가능
      app: rs-nginx
  template:
    metadata:
      labels:
        app: rs-nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```