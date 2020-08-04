- https://github.com/agronholm/sqlacodegen
- 얘 사용하면 된다.
- 말 그대로 "코드" 제너레이터다. sqlalchemy model 코드를 만들어줌.

예를 들어 터미널에서 다음과 같은 식으로 입력한다.

```bash
sqlacodegen mysql+mysqlconnector://{user_id}:{user_password}\@{host}:{port}/{database} --tables {table_name}
```

엔터치면 스트링으로 코드 쭈우우욱 뽑아줌.