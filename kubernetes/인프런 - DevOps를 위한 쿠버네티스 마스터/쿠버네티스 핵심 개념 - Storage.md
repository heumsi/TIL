# Storage

### Volume 개요와 종류

- 컨테이너가 외부 스토리지에 액세스하고 공유하는 방법
- 독립적인 쿠버네티스 오브젝트가 아니며 스스로 생성, 삭제 불가
- 각 컨테이너 파일 시스템의 불륨을 마운트하여 생성
- 볼륨의 종류
    - 임시 볼륨 (Container)
        - emptyDir
    - 로컬 볼륨 (Node)
        - hostpath
        - local
    - 네트워크 볼륨 (클라우드 종속적)
        - NFS 등등
        - gcePersistentDisk, awsEBS 등등...



### EmptyDir

같은 포드 내 컨테이너간 볼륨을 공유
컨테이너간 공유해야하는 파일이 있을 때 사용

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: count
spec:
  containers:
  - image: gasbugs/count
    name: html-generator
    volumeMounts:  # 2. container에 volume 마운트
    - mountPath: /var/htdocs  # 3. volume이 마운트되는 container 내 경로
      name: html   # 4. 아래 volume 이름과 같아야 함.
  - image: httpd
    name: web-server
    volumeMounts: 
    - name: html
      mountPath: /usr/local/apache2/htdocs 
      readOnly: true
    ports:
    - containerPort: 80 
  volumes:  # 1. emptyDir 타입의 volume 정의
  - name: html
    emptyDir: {}
```



### hostPath

컨테이너와 노드간의 볼륨을 공유.
보통 노드 모니터링 (리소스 등) 하는데에 쓰임.

```bash
# GCP 기준 fluentd 를 살펴보면 hostPath Volume을 확인할 수 있음

$ kubectl describe pod fluentd-gcp-v3.1.1-sx8fz -n kube-system
...
Volumes:
  varrun:
    Type:          HostPath (bare host directory volume)
    Path:          /var/run/google-fluentd
    HostPathType:
  varlog:
    Type:          HostPath (bare host directory volume)
    Path:          /var/log
    HostPathType:
  varlibdockercontainers:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/docker/containers
    HostPathType:
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: count
spec:
  containers:
  - image: httpd
    name: web-server
    volumeMounts:
    - mountPath: /usr/local/apache2/htdocs
      name: html
      readOnly: true
    ports:
    - containerPort: 80
  volumes:
  - name: test-volume
    hostPath: # 1. hostPath 타입의 volume 정의
      path: /data  # 2. 노드 내 마운트할 디렉토리
      type: Directory
```



### GCE 영구 스토리지

외부의 영구 디스크를 사용하기 때문에 데이터가 지속되고, 모든 포드에서 공유 가능.

```bash
# 먼저 다음 명령어로 mongodb 라는 gcePersistentDisk를 생성 

$ gcloud compute disks create --size=10GiB --zone=asia-northeast3-a mongodb
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mongodb
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - mountPath: /data/db
      name: mongodb
  volumes:
  - name: mongodb
    gcePersistentDisk:  # 1. gcePersistentDisk 타입의 volume 정의
      pdName: mongodb  # 2. gcePersistentDisk의 이름 
      fsType: ext4
```



### NFS

클러스터 외 별도 볼륨용 서버 사용. (즉 클러스터, 클라우드 벤더와는 독립적)
실습은 생략함.

