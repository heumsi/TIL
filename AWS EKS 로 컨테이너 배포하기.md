

# AWS EKS 로 컨테이너 배포하기 1



>  공식 참고 문서 
>
> https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started.html



## 0. 들어가기 전

AWS EKS 는 Elatsic Kubernetes Service 의 약자로, AWS 가 제공해주는 Kubernetes 서비스다.
이 글은 EKS 를 활용하여 컨테이너를 띄우는 과정을 다루며, 쿠버네티스로 컨테이너를 배포하는 기초적인 과정을 다룬다.
사전에 준비되어야할 것은 다음과 같다.

도커 이미지가 사전에 준비되어야 있어야 한다.



##1. VPC, 서브넷 설정

### 개요

앞으로 우리는 AWS 라는 대륙에서 컨테이너라는 상점을 세우려고 한다.
이를 위해서는 구체적으로 VPC, 서브넷 이라는게 필요한데, 간단히 비유해보면 이렇다.

- AWS : 지구에 있는 하나의 거대한 대륙. 다른 대륙으로는 GCP 가 있다.
- VPC : 대륙 내 하나의 거대한 지역이다.
- 서브넷 : 지역 내에 세워진 하나의 상가다. 여기에 상점이 입점된다.
- 노드 : 상가에 들어서는 하나의 상점이다. 
- 

### VPC

![image-20200627164728396](images/AWS EKS 로 컨테이너 배포하기/image-20200627164728396.png)



### 서브넷

![image-20200627164703635](images/AWS EKS 로 컨테이너 배포하기/image-20200627164703635.png)



- 퍼블릭 IPv4 주소 자동 할당 : 예
- IPv4 CIDR 로 가용 범위 지정



## 2. 역할 생성

### 개요





### Role for EKS

![image-20200627165256338](images/AWS EKS 로 컨테이너 배포하기/image-20200627165256338.png)

- 신뢰할 수 있는 유형의 개체 선택 : AWS 서비스
- 사용사례 선택 : EKS - Cluster



![image-20200627165435742](images/AWS EKS 로 컨테이너 배포하기/image-20200627165435742.png)



![image-20200627165645713](images/AWS EKS 로 컨테이너 배포하기/image-20200627165645713.png)



### Role for Nodegroup

![image-20200627172929787](images/AWS EKS 로 컨테이너 배포하기/image-20200627172929787.png)

- 신뢰할 수 있는 유형의 개체 선택 : AWS 서비스
- 사용사례 선택 : EC2



![image-20200627173038001](images/AWS EKS 로 컨테이너 배포하기/image-20200627173038001.png)

- 정책 필터에서 다음 정책들을 찾아서 선택
    - `AmazonEKSWorkerNodePolicy`
    - `AmazonEKS_CNI_Policy`
    - `AmazonEC2ContainerRegistryReadOnly`



![image-20200627173316338](images/AWS EKS 로 컨테이너 배포하기/image-20200627173316338.png)





> **참고 링크**
>
> https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/worker_node_IAM_role.html#create-worker-node-role



## 3. 키페어 생성

![image-20200627175748729](images/AWS EKS 로 컨테이너 배포하기/image-20200627175748729.png)

- EC2 - 키 페어



![image-20200627180252863](images/AWS EKS 로 컨테이너 배포하기/image-20200627180252863.png)



## 4. 클러스터 생성

### 개요



![image-20200627165016155](images/AWS EKS 로 컨테이너 배포하기/image-20200627165016155.png)



### 클러스터 구성

![image-20200627170014771](images/AWS EKS 로 컨테이너 배포하기/image-20200627170014771.png)

- 클러스터 서비스 역할에 아까 IAM - Role 에서 만든 `RoleForEKSCluster` 를 설정



### 네트워크 지정

![image-20200627170336235](images/AWS EKS 로 컨테이너 배포하기/image-20200627170336235.png)



### 로깅 구성

![image-20200627170416657](images/AWS EKS 로 컨테이너 배포하기/image-20200627170416657.png)



### 정리

### ![image-20200627170536727](images/AWS EKS 로 컨테이너 배포하기/image-20200627170536727.png)



## 4. 노드그룹 생성

![image-20200627171756497](images/AWS EKS 로 컨테이너 배포하기/image-20200627171756497.png)



###  노드 그룹 구성

![image-20200627180450947](images/AWS EKS 로 컨테이너 배포하기/image-20200627180450947.png)



### 컴퓨팅 구성 설정

![image-20200627180543564](images/AWS EKS 로 컨테이너 배포하기/image-20200627180543564.png)



### 조정 구성 설정

![image-20200627180620746](images/AWS EKS 로 컨테이너 배포하기/image-20200627180620746.png)



### 완료

![image-20200627180958154](images/AWS EKS 로 컨테이너 배포하기/image-20200627180958154.png)

