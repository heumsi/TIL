# Namespace

- 리소스를 각각의 분리된 영역으로 나누기 좋은 방법
- 리소스 이름은 네임스페이스 내에서만 고유 명칭 사용
- 네임스페이스 별로 리소스 접근 허용과 양도 제어 가능
- 별도로 지정해주지 않으면 `default` 네임스페이스 사용

```bash
# 네임스페이스 생성
$ kubectl create ns office

# 다음처럼 dry-run 을 이용하여 yaml 파일 생성하여 만들 수도 있음.
$ kubectl create ns office2 --dry-run -o yaml > ns-office.yaml

# 네임스페이스 조회
$ kubectl get ns

# 특정 네임스페이스에 속하는 리소스 만들기
$ kubectl create deployment nginx --image nginx --port 80 -n office

# 특정 네임스페이스에 속하는 리소스 조회
$ kubectl get all -n office

# 전체 네임 스페이스에 대한 리소스 조회
$ kubectl get all -A

# 네임 스페이스 삭제 (여기에 속한 리소스도 모두 삭제됨)
$ kubectl delete ns office
```

네임스페이스에 속하도록 리소스를 만들 때, 아래처럼 `meta.namespace` 에 적어주면 됨.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: office  # 여기다가 적어주면 됨.
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
```

