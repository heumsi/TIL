# pipeline 환경 변수 수정 후 배포해도, 반영이 안되는 이슈

### 현상

- 워크로드 들어가보면 안바뀐채로 그대로 있음.
- 워크로드 re-deploy 해도 안바뀜 ;;;
- 워크로드 삭제하고, develop 에 commit 찍어서 파이프라인 인스턴스 새로 만들어져도 answer 적용안됨 ㅋㅋㅋㅋㅋㅋㅋㅋ 뭐냐
- 후.. 워크로드. edit 들어가서 env 바꾸고 answer 하니까 팟 재배포됨. 잘 반영 됨 이제
- 진짜 모르게따 ㅋㅋㅋㅋㅋㅋ 느낌상 pipeline 에서 answer 가 덮어쓰기가 안되고 있는거 가틈.

### 원인

- 난 정말 바보다. 
- helm chart 내 ` values.yaml` 랑 rancher 내 pipeline 에서 주입해주는 answer 의 key 값이 달랐던 것
    - ex. `values.yaml` 에는 `mysqlHost` 라고 적었는데, rancher pipeline 에는 `MYSQL_HOST` 라고 적음.
    - 그니까 당연히 못찾지 ;;; 난 바보가 맞다.

### 해결

- **values.yaml 에 들어가는 키 값과 pipeline 에서 overwrite 해줄 키 값의 이름을 동일하게 넣자.**
- 헷갈리지 말고 ㅠㅠㅠ
- 그리고 참고로 values.yaml 에 들어가는 키값은 **camelCase** 로 하는게 일반적인듯 하다.
    - 고로 pipeline 에 들어가는 answer 의 키 값 역시 그대로 camelCase 따라가는게 맞다.
- 환경변수는 UPPER_CASE 로 주는게 맞고.
