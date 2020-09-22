# Service

- 포드의 문제점
    - 포드는 일시적으로 생성한 컨테이너의 집합
    - 포드 IP 주소는 지속적으로 바뀔 수 있고, 로드 밸런싱을 관리해줄 또 다른 개체가 필요
    - 이 문제를 해결하기 위해 서비스라는 리소스가 존재
- 서비스는 포드를 선택하고, 로드 밸런싱해줌.
- 서비스 타입은 기본적으로 `ClusterIP` (클러스터 내부에서만 접속 가능) 으로 정해짐.

 일반적으로 다음과 같은 리소스 템플릿을 가짐.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: http-go-svc
spec:
  selector:
    app: http-go  # pod 의 label이 app: http-go인 pod들을 선택함 
  ports:
    - protocol: TCP
      port: 80  # service의 포트
      targetPort: 8080  # pod의 포트
```

리소스 생성 후, `kubectl get svc http-go-svc -o yaml` 명령어로 만들어진 서비스를 확인해보면

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2020-09-19T07:06:41Z"
  name: http-go-svc
  namespace: default
  resourceVersion: "1244513"
  selfLink: /api/v1/namespaces/default/services/http-go-svc
  uid: 85c913e3-345b-4e63-acf5-e14ac268ae4d
spec:
  clusterIP: 10.36.10.138
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: http-go
  sessionAffinity: None   # ClientIP 로 주면, 각각의 클라이언트 IP는 특정 Pod에만 연결되도록 함. (로그인 세션 문제 등에 유용)
  type: ClusterIP  # 기본적으로 ClusterIP 타입으로 할당됨.
status:
  loadBalancer: {}
```



### 서비스를 노출하는 3가지 방법

- NodePort: 노드의 자체 포트를 사용하여 포드로 리다이렉션
- LoadBalancer: 외부 게이트웨이를 사용해 노드 포트로 리다이렉션 (L4 레벨)
- Ingress: 하나의 IP 주소를 통해 여러 서비스를 제공하는 특별한 메커니즘 (L7 레벨)



#### 1) NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: http-go-np
spec:
  type: NodePort  # type을 NodePort로 지정
  selector:
    app: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30001  # 30000-32767 포트 범위만 사용 가능
```

위 리소스를 만들고 나면 다음처럼 서비스가 떠있는 것을 확인할 수 있음.

```bash
$ kubectl get svc
NAME          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
http-go-np    NodePort    10.36.6.100   <none>        80:30001/TCP   40m
```

외부 접속은 다음처럼 가능함.

```bash
# 먼저 로컬에서 30001번 포트에 접근할 수 있도록 방화벽 규칙을 만들어 줌
$ gcloud compute firewall-rules create http-go-svc-rule --allow=tcp:30001

# 노드의 external ip 확인
$ kubectl get nodes -o wide

# curl로 container 접근
$ curl 34.64.245.242:30001
Welcome! http-go-6646dcbfc4-nm4t9
```



#### 2) LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: http-go-lb
spec:
  type: LoadBalancer  # type을 LoadBalancer로 지정
  selector:
    app: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

위 리소스를 만들고 나면 다음처럼 서비스가 떠있는 것을 확인할 수 있음.

```bash
$ kubectl get svc
NAME          TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)        AGE
http-go-lb    LoadBalancer   10.36.2.37    34.64.189.127   80:31660/TCP   56s
```

NodePort가 `31660` 으로 자동으로 생성됨.
외부 접속은 그냥 `External-IP` 값인 `34.64.189.127` 로 접근 가능

```bash
$ curl 34.64.189.127
Welcome! http-go-6646dcbfc4-nm4t9
```



#### 3) Ingress

NodePort Service가 있어야 함. (위 `.yaml` 활용)

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: http-go-ingress
spec:
  rules:
  - host: gasbugs.com  # 미리 구입한 도메인
    http:
      paths:
      - path: /  # gasbugs.com 로 http 요청에 대해서
        backend:
          serviceName: http-go-svc  # http-go-svc 80 포트로 서빙
          servicePort: 80
```

위 리소스를 만들고 나면 다음처럼 인그레스가 떠있는 것을 확인할 수 있음.

```bash
$ kubectl get ingress
NAME              HOSTS         ADDRESS   PORTS   AGE
http-go-ingress   gasbugs.com             80      13m
```

