# pipeline 설정 어떻게 할것이냐

### 1차 접근

- dev 는 develop 브랜치 push 로
- stage, prod 는 각각 release, master 브랜치 pr 로 트리거 구분.
- 이렇게 한 뒤, 클러스터별 pipeline 트리거 자체를 push / pr 로 구분.
- 그러면 develop 에 push 찍힐 때는 개발 클러스터에서 dev 버전 앱만 뜨고,
- release, master 에 pr 찍힐 때는 운영 클러스터에 stage, prod 버전 앱만 뜸.

사실 아직 안해봄. 팀장님 말 따라가고 있다...

근데 아직 운영 클러스터에 파이프라인이 안보여 ㅠ 개발엔 보이는데 ...



### 이게 정답인거 같은 접근

- 잘 모르겠어서 랜처를 그래도 잘 아신다는 TC 팀의 레디에게 찾아갔다. 
- 다음과 같은 모범답안을 주심
    - **먼저 rancher pipeline 은 develop 브랜치에 대해서만 만들면 된다.**
        - pipeline 을 rancher 웹페이지를 통해 만들면 해당 브랜치에 pipeline yaml 파일이 생긴다.
        - 그리고 어차피 develop -> release -> master 로 PR 날리며 머지해나갈거기 때문에 이 pipeline yaml 파일은 나머지 두 브랜치에도 순차적으로 복사된다. 즉 머지해나가는 과정에서 다른 브랜치도 파이프라인이 만들어진다는 뜻
    - **Pipeline 의 과정은 크게 clone -> build -> deploy 로 나뉜다.**
        - clone 은 그냥 무조건 맨 처음 시작으로, 사실 그냥 박혀있음.
        - build 는 Dockerfile 빌드한 뒤, 빌드된 도커 이미지를 Remote Repository 에다가 푸시하는건데, develop 브랜치에서만 하면 된다.
            release, master 에서는 빌드할 필요 없음. (어차피. 코드는 develop 에서만 수정되므로)
        - deploy 는 Remote Repository 에서 이미지를 풀받은 뒤, helm chart 를 쿠버네티스에 배포하는 과정이다.
            이건 develop, release, master 에 모두 해줘야 한다.
    - **파이프라인의 트리거는 모두 push 로 한다**.
- 참고로, 파이프라인은 클러스터 상관없이 전역적으로 1개고
- 클러스터 별로 파이프라인의 트리거를 다르게 줄 수 있다.
    - 예를 들어, dev 클러스터는 해당 레포가 push 될 때 파이프라인 트리거를 주고
    - prod 클러스터는 pull request 될 때 트리거를 준다던가...
    - 근데 dev 든 prod 든 클러스터 상관없이 모두 push로 한다고 함.



### 여담

여담으로, 레디의 말로는 CI/CD 툴로 랜처를 잘 안쓴다고 한다.
간단하게는 괜찮은데 기능이 별로 없고 자유도가 부족하다랄까... 암튼 살짝 엉성하다고 하셨다.
자기 팀은 [버디웍스](https://buddy.works/) 를 쓰신다고 하니.. 나도 추후에 살펴봐야겠다.