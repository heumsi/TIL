# Network

### 쿠버네티스 네트워크 모델

- 한 포드에 있는 다수의 컨테이너끼리 통신
- 포드끼리 통신
- 포드와 서비스 사이의 통신
- 외부 클라이언트와 서비스 사이의 통신



### Core DNS

- Core DNS 서비스를 생성하면, DNS 엔트리가 생성
    - ex. `kube-dns.kube-system`
- 이 엔트리는 `<서비스 이름>.<네임스페이스 이름>.svc.cluster.local` 형식
    - 뒤에 `sv.cluster.local` 은 생략 가능
    - ex. `http-go-svc.(default 생략)`
- 내부에서 DNS 서버 역할을 하는 포드가 존재
    - ex. `kube-proxy-gke-cluster-1-default-pool-b3ceab79-bj4x.kube-system`
- pod에서도 Subdomian을 사용하면 DNS 사용가능

