# AWS EKS 로 컨테이너 배포하기 2



## 1. awscli 설정

### awscli 설치

```
aws --version
```

> 버전 1.18.61 이상 또는 버전 2.0.14 이상이 설치되어 있지 않으면 AWS CLI 버전 2를 설치합니다.



```
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```



### awscli configure 설정

![image-20200627183343873](/Users/heumsi/Library/Application Support/typora-user-images/image-20200627183343873.png)







![image-20200627183408636](/Users/heumsi/Library/Application Support/typora-user-images/image-20200627183408636.png)







![image-20200627183427625](/Users/heumsi/Library/Application Support/typora-user-images/image-20200627183427625.png)



```
$ aws configure
AWS Access Key ID [None]: AKIAQW2MUUWURFPET6MF
AWS Secret Access Key [None]: {시크릿 키 입력}
Default region name [None]: ap-northeast-2
Default output format [None]: 
```





## 2. kubectl 설치

```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/darwin/amd64/kubectl
```

```
chmod +x ./kubectl
```

```
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
```

```
kubectl version --short --client
```





## 3. kubectl 과 aws 연동

```
aws eks --region ap-northeast-2 update-kubeconfig --name tutorial-cluster
```

```
Added new context arn:aws:eks:ap-northeast-2:049015989673:cluster/tutorial-cluster to /Users/heumsi/.kube/config
```



```
kubectl get svc
```

```
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   15m
```

