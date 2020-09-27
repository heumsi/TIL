# RBAC를 활용한 롤 기반 액세스 컨트롤

- 역할 기반 액세스 제어 (RBAC)
    - 기업 내에서 개별 사용자의 역할을 기반으로 컴퓨터, 네트워크 리소스에 대한 액세스를 제어
    - `rbac.authorization.k8s.io` API 를 사용하여 정의
    - 권한 결정을 내리고, 관리자가 Kubernetes API를 통해 정책을 동적으로 구성
    - RBAC를 사용하여 룰을 정의하려면, apiserver에 `--authorization-mode=RBAC` 옵션이 필요
- RBAC 관련 4가지 리소스
    - 네임 스페이스에 종속적
        - Role
        - RoleBinding
    - 네임 스페이스에 종속적이지 않음. (클러스터 전역)
        - ClusterRole
        - ClusterRoleBinding
- Role / RoleBinding
    - Role은 "누가"하는 것인지는 정의하지 않고, 롤만을 정의
    - RoleBinding은 "누가"하는 것인지만 정하고 롤은 정의하지 않음

![What is RBAC in Kubernetes?. RBAC stands for Role-Based Access… | by Daniel  Chernenkov | Medium](https://miro.medium.com/max/624/1*j8UMSaN5G7KekoOgLEVmsg.png)

*(출처 : https://medium.com/@danielckv/what-is-rbac-in-kubernetes-c54457eff2dc)*



먼저 다음처럼 ServiceAccount를 만들고,

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: heumsi
```

Role을 만든다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

그리고 이 둘을 연결(Binding)하는 RoleBinding을 만든다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: heumsi-pod-reader-binding
subjects:
- kind: ServiceAccount
  name: heumsi
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

이렇게 권한이 부여된 ServiceAccount는 다음과 같이 쓸 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-reader
spec:
  serviceAccountName: heumsi # 여기! 
  containers:
  - name: pod-reader
    image: busybox
    ports:
    - containerPort: 8080
```

