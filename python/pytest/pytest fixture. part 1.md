# Pytest Fixture. Part 1

- pytest 를 사용하는 주된 이유임.
- 자주 사용되는 코드 조각들을 재 사용할 수 있도록 미리 만들어두고 씀. 이를 Fixture 라고 함.

<br>

## 1. 기본 사용법

### 간단한 예제

```python
# test_func.py

import pytest

@pytest.fixture
def data():  # fixture 정의
    return [1, 2, 3]

def test_func(data):  # 파라미터로 받을 수 있음.
    assert data == [1, 2, 3]
```

```bash
$ pytest test_func.py
=============================== test session starts ===============================
platform darwin -- Python 3.7.2, pytest-5.4.3, py-1.8.0, pluggy-0.13.1
rootdir: /Users/heumsi/Desktop/dev/playground
plugins: openfiles-0.3.2, arraydiff-0.3, doctestplus-0.3.0, remotedata-0.3.1
collected 1 item                                                                  

test_func.py .                                                              [100%]

================================ 1 passed in 0.01s ================================
```

- 함수 위에 `@pytest.fixture` 데코레이터를 붙여서 fixture 를 정의함.
- 이제 이 함수의 이름과 반환 값을 기준으로 fixture 가 생성.
    - 즉 `data = [1, 2, 3]` 으로 정의되고, 테스트 함수에서 `data` 를 파라미터로 이를 받을 수 있음.
    - **fixture 는 함수 이름으로 정의되고, 테스트 함수에서 받을 때도 이 이름을 사용해야 함.**
- 일종의 DI (Dependency Injection) 기법임.
- 테스트 코드 곳곳에서 자주 사용되는 데이터 조각들은 이렇게 fixture 화 하여 재사용할 수 있음.

<br>

## 2. fixture의 속성들

### scope

`scope` 파라미터로, fixture 인스턴스의 유효 범위를 지정할 수 있다.
일단, fixture 를 파라미터로 받아올 때 각 id 를 찍어보자.

```python
# test_func.py

import pytest

@pytest.fixture
def data():
    return [1, 2, 3]

def test_func(data):
    print(id(data))  # fixture 의 id 를 찍어본다.  
    assert False  # stdout 을 보기 위해 일부러 실패하도록 함.

def test_func2(data):
    print(id(data))  # 여기도!
    assert False
```

- 결과를 보기 위해서 일부러 `assert False` 를 넣어주었다.
    - **pytest 는 테스트에 실패해야 `print` 등의 stdout 을 볼 수 있다.**
    - [이 링크](https://stackoverflow.com/questions/24617397/how-to-print-to-console-in-pytest) 참고 

```bash
$ pytest test_func.py
=============================== test session starts ===============================
platform darwin -- Python 3.7.2, pytest-5.4.3, py-1.8.0, pluggy-0.13.1
rootdir: /Users/heumsi/Desktop/dev/playground
plugins: openfiles-0.3.2, arraydiff-0.3, doctestplus-0.3.0, remotedata-0.3.1
collected 2 items                                                                 

test_func.py FF                                                             [100%]

==================================== FAILURES =====================================
____________________________________ test_func ____________________________________

data = [1, 2, 3]

    def test_func(data):
        print(id(data))  # fixture 의 id 를 찍어본다.
>       assert False  # stdout 을 보기 위해 일부러 실패하도록 함.
E       assert False

test_func.py:13: AssertionError
------------------------------ Captured stdout call -------------------------------
4500174728
___________________________________ test_func2 ____________________________________

data = [1, 2, 3]

    def test_func2(data):
        print(id(data))  # 여기도!
>       assert False
E       assert False

test_func.py:18: AssertionError
------------------------------ Captured stdout call -------------------------------
4527323976
============================= short test summary info =============================
FAILED test_func.py::test_func - assert False
FAILED test_func.py::test_func2 - assert False
================================ 2 failed in 0.10s ================================
```

- 위에서 `# 여기 주목 1`, `# 여기 주목 2` 부분을 보면, 파라미터로 fixture 를 받는 객체가 각각 다른 인스턴스임을 알 수 있다.
- 이를 fixture 의 scope 라고 하는데, **기본적으로 scope는 `function` 이다.** 즉, fixture 는 함수 단위로 인스턴스를 다르게 갖는다.
- **scope 는 `function`, `class`, `package`,  `module`, `package` , `session` 이 있다.**

<br>

fixture 의 scope 는 다음과 같이 `scope` 파라미터로 설정할 수 있다.  
위 코드를 아래와 같이 수정하자.

 ```python
@pytest.fixture(scope="module")  # scope 를 module 단위로 정의했다.
def data():  # fixture 정의
    return [1, 2, 3]
 ```

이제 이 모듈 `test_func.py` 에서 `data` fixture 는 단일 인스턴스다.  
다시 pytest 를 실행해보자.

```bash
$ pytest test_func.py
=============================== test session starts ===============================
platform darwin -- Python 3.7.2, pytest-5.4.3, py-1.8.0, pluggy-0.13.1
rootdir: /Users/heumsi/Desktop/dev/playground
plugins: openfiles-0.3.2, arraydiff-0.3, doctestplus-0.3.0, remotedata-0.3.1
collected 2 items                                                                 

test_func.py FF                                                             [100%]

==================================== FAILURES =====================================
____________________________________ test_func ____________________________________

data = [1, 2, 3]

    def test_func(data):
        print(id(data))  # fixture 의 id 를 찍어본다.
>       assert False  # stdout 을 보기 위해 일부러 실패하도록 함.
E       assert False

test_func.py:13: AssertionError
------------------------------ Captured stdout call -------------------------------
4569146568
___________________________________ test_func2 ____________________________________

data = [1, 2, 3]

    def test_func2(data):
        print(id(data))  # 여기도!
>       assert False
E       assert False

test_func.py:18: AssertionError
------------------------------ Captured stdout call -------------------------------
4569146568
============================= short test summary info =============================
FAILED test_func.py::test_func - assert False
FAILED test_func.py::test_func2 - assert False
================================ 2 failed in 0.10s ================================
```

두 테스트 함수에서 쓰는 `data` fixture 는 같은 인스턴스임을 알 수 있다.

scope 의 우선순위는 어떻게 될까? 는 [여기](https://docs.pytest.org/en/stable/fixture.html#order-higher-scoped-fixtures-are-instantiated-first) 참고   
이 글에서는 최대한 기초만 다루려고 하는거고... 절대 귀찮아서 안쓴게 아님...

<br>

### autouse

`autouse` 파라미터로 항상 실행되는 fixture 를 만들 수 있다.

```python
import pytest

@pytest.fixture(autouse=True)  # autouse=True 로 주었다. 기본 값은 False 임.
def function_autouse():
    print("scope function with autouse")

def test_autouse():
    assert False  # stdout 을 보기 위해 일부러 실패하도록 함.
```

```bash
pytest test_func.py
=============================== test session starts ===============================
platform darwin -- Python 3.7.2, pytest-5.4.3, py-1.8.0, pluggy-0.13.1
rootdir: /Users/heumsi/Desktop/dev/playground
plugins: openfiles-0.3.2, arraydiff-0.3, doctestplus-0.3.0, remotedata-0.3.1
collected 1 item                                                                  

test_func.py F                                                              [100%]

==================================== FAILURES =====================================
__________________________________ test_autouse ___________________________________

    def test_autouse():
>       assert False  # stdout 을 보기 위해 일부러 실패하도록 함.
E       assert False

test_func.py:10: AssertionError
------------------------------ Captured stdout setup ------------------------------
scope function with autouse
============================= short test summary info =============================
FAILED test_func.py::test_autouse - assert False
================================ 1 failed in 0.09s ================================
```

`function_autouse` 을 파라미터로 받지도, 따로 실행하지도 않았는데, 알아서 실행되었음을 알 수 있다.

<br>

### yield

fixture 내 `yield` 를 통해 setup, teardown 의 효과를 낼 수 있다.

```python
# test_func.py

import pytest

@pytest.fixture
def function_yield():
    print("Executing setup code")
    yield "Heumsi"
    print("Executing teardown code")

def test_yield(function_yield):
    print(function_yield)
    assert False  # stdout 을 보기 위해 일부러 실패하도록 함.
```

```bash
$ pytest test_func.py
=============================== test session starts ===============================
platform darwin -- Python 3.7.2, pytest-5.4.3, py-1.8.0, pluggy-0.13.1
rootdir: /Users/heumsi/Desktop/dev/playground
plugins: openfiles-0.3.2, arraydiff-0.3, doctestplus-0.3.0, remotedata-0.3.1
collected 1 item                                                                  

test_func.py F                                                              [100%]

==================================== FAILURES =====================================
___________________________________ test_yield ____________________________________

function_yield = 'Heumsi'

    def test_yield(function_yield):
        print(function_yield)
>       assert False  # stdout 을 보기 위해 일부러 실패하도록 함.
E       assert False

test_func.py:13: AssertionError
------------------------------ Captured stdout setup ------------------------------
Executing setup code
------------------------------ Captured stdout call -------------------------------
Heumsi
---------------------------- Captured stdout teardown -----------------------------
Executing teardown code
============================= short test summary info =============================
FAILED test_func.py::test_yield - assert False
================================ 1 failed in 0.10s ================================
```

<br>



## 3. fixture 따로 관리하기

fixture 는 `conftest.py` 라는 파일에서 별도로 관리할 수 있다.
별건 없고, 테스트 코드를 짜다보면 여러 테스트 코드에 공통의 fixture가 쓰일 수 있는데, 이렇게 여러군데 사용되는 fixture 를 모아두는 것이다.

아래 예를 보자.

```python
# conftest.py

import pytest

@pytest.fixture
def my_name():
    return "heumsi"
```

```python
# test_my_name.py

import pytest

def test_my_name(my_name):
    assert my_name == "heumsi"
```

```bash
$ pytest test_func.py
=============================== test session starts ===============================
platform darwin -- Python 3.7.2, pytest-5.4.3, py-1.8.0, pluggy-0.13.1
rootdir: /Users/heumsi/Desktop/dev/playground
plugins: openfiles-0.3.2, arraydiff-0.3, doctestplus-0.3.0, remotedata-0.3.1
collected 1 item                                                                  

test_func.py .                                                              [100%]

================================ 1 passed in 0.01s ================================
```

fixture 가 테스트 코드 `test_my_name.py` 와 별도의 파일 `conftest.py` 에 있음에도, 테스트 코드에 fixture 가 잘 주입된 것을 볼 수 있다. (심지어 `import` 도 안했다. )

이렇듯, 테스트 코드와 fixture 를 분리해놓을 수 있으며, `conftest.py` 는 경로에 따라 여러 개를 둘 수 있다.
다만 그 경로가 어디에 위치해있으냐에 따라 fixture 의 인식 범위가 정해진다.

예를 들어 다음과 같은 파일 구조를 가지고 있다고 하자.

```basgh
├── conftest.py
├── test_a
│   ├── conftest.py
│   └── test_func.py
└── test_b
    ├── conftest.py
    └── test_func.py
```

- `test_a/test_func.py` 와 `test_b/test_func.py` 는 `/conftest.py` 에 정의된 fixture를 인식할 수 있다.
- 하지만 `test_b/test_func.py` 은 `test_a/conftest.py` 의 fixture 를 인식할 수 없다.
- 마찬가지로 `test_a/test_func.py` 는 `test_b/conftest.py` 의 fixture 를 인식할 수 없다.

fixture 를 구조적으로 잘 설계해야함의 필요성을 느끼게 된다.



## 4. Factory 로의 활용

아래와 같이 객체를 생성하는 factory function 을 내보내는 fixture 를 만들 수도 있다.

```python
@pytest.fixture
def make_customer_record():
    def _make_customer_record(name):
        return {"name": name, "orders": []}

    return _make_customer_record


def test_customer_records(make_customer_record):
    customer_1 = make_customer_record("Lisa")
    customer_2 = make_customer_record("Mike")
    customer_3 = make_customer_record("Meredith")
```

<br>

## 5. 참고 및 출처

- https://pythontesting.net/framework/pytest/pytest-fixtures-easy-example/
- https://medium.com/better-programming/understand-5-scopes-of-pytest-fixtures-1b607b5c19ed
- https://docs.pytest.org/en/stable/fixture.html#parametrizing-fixtures