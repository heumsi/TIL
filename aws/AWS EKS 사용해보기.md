

# AWS EKS 사용해보기



## 0. 들어가기 전

AWS EKS 는 Elatsic Kubernetes Service 의 약자로, AWS 가 제공해주는 Kubernetes 서비스다.
이 글은 EKS 를 사용해보는 기초적인 과정을 다룬다. 크게 다음의 내용을 다룬다.

- EKS 사용 전 설정해야 하는 것들 (네트워크, 키페어 등등)
- EKS 만들기
- 클러스터 (Node Group) 만들기
- 로컬의 kubectl 과 연동하기

EKS 를 쓴다는 것은 결국 쿠버네티스 환경에 컨테이너를 배포한다는 목적인 건데, 구체적인 배포 과정은 다른 글에서 적으려고 한다.
이 글에서는 딱 kubectl 과 연동하는 부분까지만 다룬다. 



> **[AWS 공식 참고 문서]**
>
> 공식참고 문서는 [여기](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started.html)에 있다.
> AWS 의 공식문서는 되게 세세하게 설명되어 있긴 하지만, 동시에 너무 많은 정보를 보여줘 가독성이 떨어지기도 한다.
> 이 글이 공식문서보다 잘 읽혔으면 좋겠다.
>
> 클라우드 벤더사의 제품은 빠르게 변화하기 때문에 .. 그래도 역시 공식 문서를 참고하는게 제일 정확하고 좋긴하다.



##1. VPC, 서브넷 설정

앞으로 우리는 AWS 라는 대륙에서 컨테이너라는 상점을 세우려고 한다.
이를 위해서는 네트워크 세계에서는 VPC, 서브넷 이라는게 필요한데, 간단히 비유해보면 이렇다.

- AWS : 하나의 거대한 대륙. 다른 대륙으로는 GCP 가 있다.
- VPC : 대륙 안에 의도적으로 구분해놓은 하나의 거대한 지역이다.
- 서브넷 : VPC 지역 내에 세워진 하나의 상가다. 이 안에는 컨테이너라는 상점이 입점할 수 있는 공간들이 있다.

보통 개발팀은 회사의 VPC 나 서브넷에 대하여 인프라팀에게 문의한다.
어떤 규칙을 가지고 만들면 좋을지, 혹은 만들어진 곳이 있다면 어떤 것을 쓰면 좋을 지 등 ...

VPC 는 하나의 EKS 와 매핑된다. 즉 하나의 클러스터 환경은 하나의 VPC 를 사용한다.
서브넷은 VPC 내의 땅을 여러개로 나누어 먹는다. 즉 하나의 클러스터 환경, VPC 는 여러개의 서브넷을 가진다.



> **AWS VPC 에 대해 잘 써놓은 글**
>
> 개인적으로 [이 글](https://medium.com/harrythegreat/aws-%EA%B0%80%EC%9E%A5%EC%89%BD%EA%B2%8C-vpc-%EA%B0%9C%EB%85%90%EC%9E%A1%EA%B8%B0-71eef95a7098)이 VPC 와 서브넷 등 네트워크 개념들을 빠르게 이해하는데 제일 좋았다.
> 나같이 잘 모르는 사람들은 꼭 보면 좋을 듯



### VPC

**서비스 > VPC > VPC** 에 가보면 다음 화면을 볼 수 있다.

![image-20200627164728396](images/AWS EKS 사용해보기/image-20200627164728396.png)

기본적인 VPC 가 이미 설정되어 있다.
계정을 처음 만들었다면 기본적으로 이렇게 VPC 하나가 만들어져있을 것이다. 
나는 이 VPC 를 사용하겠다.
만약 회사에서 작업 중이라면, 인프라팀에게 어떤 VPC 를 사용해야 할지 물어보는 것이 좋다.



### 서브넷

**서비스 > VPC > 서브넷** 을 누르면 등장 하는 화면이다. 

![image-20200627164703635](images/AWS EKS 사용해보기/image-20200627164703635.png)

기본적으로 위 처럼 3개의 서브넷이 만들어져있다.
이 때 다음 항목들이 `퍼블릭 IPv4 주소 자동 할당 : 예` 로 잘 설정되어있나 확인하자.
`아니오` 로 되어있을 경우, EKS - kubectl 연동이 작동하지 않는다.

나는 이 기본적인 서브넷 3개를 사용하겠다.
마찬가지로 회사에서 작업 중이라면 어떤 서브넷을 사용해야하는지 인프라팀에게 문의하자.



## 2. 역할 생성

AWS 에서 각각의 인스턴스는 역할(Role) 을 갖는다.
그리고 역할은 정책이라는 일종의 권한 목록을 담고 있다.
예를 들어, EKS 는 EC2 인스턴스를 동적으로 만들고 지우는데 이렇게 할 수 있는 권한을 EKS 의 역할로 주어야 한다.



### Role for EKS

**서비스 > IAM > 역할**로 들어와 역할 만들기 버튼을 누르면 다음 화면이 뜬다.

![image-20200627165256338](images/AWS EKS 사용해보기/image-20200627165256338.png)

다음 항목들을 클릭한다.

- 신뢰할 수 있는 유형의 개체 선택 : AWS 서비스
- 사용사례 선택 : EKS - Cluster



![image-20200627165435742](images/AWS EKS 사용해보기/image-20200627165435742.png)

이전 화면에서 `EKS - Cluster` 를 클릭했으면 위 처럼 `AmazonEKSClusterPolicy` 라는 정책이 알아서 잡힌다. 
그대로 넘어간다.



![image-20200627165645713](images/AWS EKS 사용해보기/image-20200627165645713.png)

나는 "RoleForEKSCluster" 라는 역할의 이름을 주었다.
검토를 다 했으면 역할 생성을 완료하자.



### Role for Nodegroup

이번엔 클러스터 내에 들어가는 노드그룹에 대한 역할을 생성한다.

![image-20200627172929787](images/AWS EKS 사용해보기/image-20200627172929787.png)

다음 항목들을 클릭한다.

- 신뢰할 수 있는 유형의 개체 선택 : AWS 서비스
- 사용사례 선택 : EC2



![image-20200627173038001](images/AWS EKS 사용해보기/image-20200627173038001.png)

- 정책 필터에서 다음 정책들을 찾아서 선택한다.
    - `AmazonEKSWorkerNodePolicy`
    - `AmazonEKS_CNI_Policy`
    - `AmazonEC2ContainerRegistryReadOnly`



![image-20200627173316338](images/AWS EKS 사용해보기/image-20200627173316338.png)



나는 "RoleForEKSNodeGroup" 이라는 이름을 주었다.



> **노드그룹 Role 에 대한 참고 링크**
>
> 노드그룹은 일반적으로 EKS 내에 들어가므로, 역할 생성의 첫 페이지의 사용사례에서 `EKS - Nodegroup` 을 선택해야할 것 같다.
> 이게 꽤 헷갈려서 나도 삽질을 좀 했었는데, [이 참고 링크](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/worker_node_IAM_role.html#create-worker-node-role)를 보면 그렇지 않다.
> 결과적으로 `EKS - Nodegroup` 이 아닌 `EC2` 를 선택해야 한다.



## 3. 키페어 생성

각 노드에 `ssh` 로 접근하기 위해서는 키페어를 생성해야 한다.

**서비스 > EC2 > 키 페어** 페이지로 들어간다.

![image-20200627175748729](images/AWS EKS 사용해보기/image-20200627175748729.png)

키 페어 생성 버튼을 누른다.



![image-20200627180252863](images/AWS EKS 사용해보기/image-20200627180252863.png)

나는 "aws_heumsi" 라는 이름으로 만들었다.
파일 형식은 `pem` 을 선택한다.

키 페어 생성 버튼을 누르면 로컬로 `.pem` 파일이 다운로드 되는데, `~/.ssh` 안에다가 넣어두자.



## 4. 클러스터 생성

이제 본격적으로 EKS 를 생성해본다.
**서비스 > Elastic Kubernetes Service (EKS)** 페이지로 들어간다.

![image-20200627165016155](images/AWS EKS 사용해보기/image-20200627165016155.png)

클러스터 이름을 적어주고 다음 단계 버튼을 누른다.



### 클러스터 구성

![image-20200627170014771](images/AWS EKS 사용해보기/image-20200627170014771.png)

클러스터 서비스 역할에 아까 IAM > Role 에서 만든 `RoleForEKSCluster` 를 설정해준다.



### 네트워크 지정

![image-20200627170336235](images/AWS EKS 사용해보기/image-20200627170336235.png)

VPC 와 서브넷을 설정해준다.
보안 그룹은 따로 설정안했었는데, VPC 나 서브넷과 같이 계정에 기본적으로 존재하는 보안그룹이 있을 것이다.
나는 이렇게 기본 보안그룹을 사용했다. (따로 설정안했으니 이거 하나밖에 없음...)

회사라면 인프라팀에게 어떤 보안 그룹을 사용해야할지 문의해야할 듯.



### 로깅 구성

![image-20200627170416657](images/AWS EKS 사용해보기/image-20200627170416657.png)

로그는 모두 비활성화 했다.
다음 버튼을 누르고 검토 페이지에서 검토한 뒤 생성을 완료하자.



![image-20200627170536727](images/AWS EKS 사용해보기/image-20200627170536727.png)

여기까지하고 나면, 아직 `생성 중` 이라는 상태로 EKS 가 만들어지고 있을 것이다.
10분 정도 걸렸던 듯 하다.
생성 완료될 때까지 커피 한 잔 마시러 갈 겸 기다려보자.



## 4. 노드그룹 생성

이제 클러스터 내에 들어갈 노드들을 만들어줄 때다.

EKS 생성이 완료되면, 상태가 `활성` 으로 바뀌고 컴퓨팅 탭에서 다음 처럼 노드 그룹을 추가할 수 있는 버튼이 활성화 된다.

![image-20200627171756497](images/AWS EKS 사용해보기/image-20200627171756497.png)

노드 그룹 추가 버튼을 눌러주자.



###  노드 그룹 구성

![image-20200627180450947](images/AWS EKS 사용해보기/image-20200627180450947.png)

굳이 별다른 설명이 없어도 될 듯.. 

- IAM 역할
- 서브넷
- ssh 키페어

모두 이전 과정에서 우리가 만들어 놓은 것이다. 그대로 채워주자.



### 컴퓨팅 구성 설정

![image-20200627180543564](images/AWS EKS 사용해보기/image-20200627180543564.png)

돈을 아끼기 위해 `t3.micro` 로 만들었다.
돈 많으신 분들은 더 넉넉히 쓰셔도 될 듯.



### 조정 구성 설정

![image-20200627180620746](images/AWS EKS 사용해보기/image-20200627180620746.png)

오토 스케일링을 고려한 조정 항목이다.
실전에서는 다르게 조정하기로 하고, 지금은 튜토리얼이니까 1, 3, 2 로 두었다.

다음 버튼을 눌러 최종 검토를 하고 생성 과정을 완료하자.



![image-20200627180958154](images/AWS EKS 사용해보기/image-20200627180958154.png)

여기까지 다 하고나면 다음처럼 노드 그룹이 생성되고 상태가 활성화 될 때 까지 기다리면 된다.
EKS 생성부터 클러스터에 들어가는 노드 생성까지 완료하였다.



## 5. awscli 설치 및 로그인

### awscli 설치

이제 이렇게 만든 EKS 를 로컬 CLI 환경에서 접근하기 위한 준비해보자.
다음 커맨드로 으로 자신의 `awscli` 버전을 확인한다. 

```
aws --version 
```

"1.18.61 이상 또는 버전 2.0.14 이상이 설치되어 있지 않으면 AWS CLI 버전 2를 설치합니다." 라는 공식문서의 권고사항이 있다.
만약 위 커맨드에 어떠한 응답이 없다면 다음 커맨드로 `awscli` 를 설치한다.

```
aws --version
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```



### awscli 로그인

설치가 완료됐으면 다음 커맨드로 awscli 에 로그인한다.

```
$ aws configure
AWS Access Key ID [None]: AKIAQW2MUUWURFPET6MF
AWS Secret Access Key [None]: {시크릿 키 입력}
Default region name [None]: ap-northeast-2
Default output format [None]: 
```

`AWS Access Key ID` 와 `AWS Secret Access Key` 는 IAM 계정에 할당되어 있는 키들이다. 이전에 "AWS. IAM 계정으로 시작하기" 글에서 IAM 계정을 만들 때 따로 적어두라고 했던 값들이다.
잘 모르겠다면 다음처럼 AWS 웹 콘솔 페이지로 가서, 서비스 > IAM > 사용자 > 보안 자격 증명 페이지로 가서 확인해보자.

![image-20200627183427625](images/AWS EKS 로 컨테이너 배포하기/image-20200627183427625.png)

`Default region name` 는 클러스터를 만든 지역인 `ap-northeast-2` (서울) 로 두었다. 
`Default output format` 는 아무것도 입력하지 않고 그대로 두었다.



## 6. kubectl 설치 및 연결

### kubectl 설치

이제 EKS 에 커맨드를 내릴 수 있는 `kubectl` 을 설치해보자.
이미 설치되어있는 사람은 이 과정을 패스하면 된다.

```curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/darwin/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
kubectl version --short --client
```



### Kubectl - EKS 연결

다음 커맨드로 kubectl 과 aws 에서 만들어놓은 EKS 를 연결할 수 있다.

```
aws eks --region ap-northeast-2 update-kubeconfig --name tutorial-cluster
```

`--region` 인자 값으로 EKS를 만든 지역을 입력해주면 되고,
`--name` 인자 값으로 EKS의 이름을 입력해주면 된다.

커맨드 실행 후 다음 결과가 뜨면 정상적으로 된 것이다.

```
Added new context arn:aws:eks:ap-northeast-2:049015989673:cluster/tutorial-cluster to /Users/heumsi/.kube/config
```



### 연결 확인

자 이제 `kubectl` 로 AWS EKS 에 잘 연결되었는지 확인해보자.

```
kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   15m
```

결과 값이 잘 나오는 것을 통해, 잘 연결되어 있는 것을 확인할 수 있다.