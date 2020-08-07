- https://github.com/agronholm/sqlacodegen
- 얘 사용하면 된다.
- 말 그대로 "코드" 제너레이터다. sqlalchemy model 코드를 만들어줌.

예를 들어 터미널에서 다음과 같은 식으로 입력한다.

```bash
sqlacodegen mysql+mysqlconnector://{user_id}:{user_password}\@{host}:{port}/{database} --tables {table_name}
```

엔터치면 스트링으로 코드 쭈우우욱 뽑아줌.

- @ 앞에 이스케이프문 `\`  이 들어가 있음. 공식문서에는 없었는데, 이스케이프문 있어야 돌드라. 
- 직접 `@dataclass` 달아주고, 어노테이션 붙여주면 `dataclass` 형태로 쓸 수 있긴 함.
- 근데 이 마저도 그냥 `codegen` 이 다 해주면 좋을텐데... 아직 그런 기능은 없음
- (그래서 `__repr__`) 을 따로 만들어줘야 함. 이런거 좀 같이 해주지 ...
- sqlalchemy 현재 1점대 인데, 2점대에서 뭔가 이와 관련된 작업을 할거라는 이슈가 있었음.

---

> 아래 내용은 못씁니다.

`__repr__` 은 유닛 테스트 구현 시에 거의 필수적인데 ㅠ  일일이 annotation 달아주면서 `dataclass` 를 쓸 수는 없어서... 방법을 찾아보다가... 이런걸 발견.


https://github.com/manicmaniac/sqlalchemy-repr

```python
from sqlalchemy_repr import RepresentableBase

Base = declarative_base(cls=RepresentableBase)
```

이 부분만 넣어주면 알아서 `__repr__` 이 주입된다.. 후후 
무려 2016년에 만들어지고. `0.0.2` 버전이라 공식적으로 절대 사용할만한 애는 아니지만 ... ㅋㅋㅋ 그래도 일단은 급한대로 써봅니다.... ~~해결 완료~~

---

라고 위에 써놨으나 해결 안됐다. 얘 `repr` 찍어보니까 인스턴스 생성 재현이 불가능하게 만들어놨네. 왜 이렇게 만들어놨을까............

결국 sqlalchemy 에서 사실 `dataclass` 형태로 인스턴스를 만들어주는게 해답인거 같은데, 지금 버전대에서는 지원이 안되는거 같고.. (sqlalchemy 업데이트 좀 느린거 같은데, 계속 써야되나 싶다)
쩔수 없이 그냥 내가 직접 `dataclass` 랑 어노테이션 달아준다 ㅠ