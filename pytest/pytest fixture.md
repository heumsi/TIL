# Fixture

- pytest 를 사용하는 이유임.
- 자주 사용되는 코드 조각들을 재 사용할 수 있도록 미리 만들어두고 씀. 이를 Fixture 라고 함.

```python
@pytest.fixture
def test(a, b):
    print("hi")
    return a, b


@pytest.mark.parametrize('test', [('aaaaa', 'bbbbb')])
def test_a(test):
    print(type(test))
    # print("test_a", test)
```

