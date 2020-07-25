# Pytest Fixture. Part 2



## 1. 좀 더 영리하게 테스트 코드 짜기 (Parametrized tests)

먼저 다음 테스트 코드를 보자.

```python
# get_gender.py

def get_gender(name):
    title = name.split()[0]
    if title in ("Mr."):
        return "man"
    elif title in ("Ms.", "Miss.", "Mrs."):
        return "woman"
```

```python
# test_get_gender.py

from get_gender import get_gender

data = [
    ("Mrs. Claire", "woman"), 
    ("Mr. Jay", "man")
]  # (입력 값, 기대 값) 을 리스트에 담는다.

def test_get_gender(name, expected):
    for input_value, expected in data: 
    	assert expected == get_gender(input_value)
```

일반적인 테스트 코드라면 이렇게 `for` 구문을 활용하여 여러 입력에 대한 테스트를 시도해볼 수 있다.  
하지만 pytest 에서 제공해주는 `@pytest.mark.parametrize` 를 이용하면 좀 더 영리하게 테스트 코드를 만들 수 있다.

테스트 코드를 바꾼 다음 예제를 보자.

```python
# test_get_gender.py

from get_gender import get_gender

data = [
    ("Mrs. Claire", "woman"),
    ("Mr. Jay", "man")
]  # (입력 값, 기대 값) 을 리스트에 담는다.

@pytest.mark.parametrize("name, expected", data)  # 여기 주목
def test_get_gender(name, expected):
    assert expected == get_gender(name)
```

```bash
$ pytest test_func.py
=============================== test session starts ===============================
platform darwin -- Python 3.7.2, pytest-5.4.3, py-1.8.0, pluggy-0.13.1
rootdir: /Users/heumsi/Desktop/dev/playground
plugins: openfiles-0.3.2, arraydiff-0.3, doctestplus-0.3.0, remotedata-0.3.1
collected 2 items                                                                 

test_func.py ..                                                             [100%]

================================ 2 passed in 0.02s ================================
```

주목해야할 부분은 `@pytest.mark.parametrize(...)` 부분이다.

- 첫 번째 인자 `"name, expected"` 는 테스트 함수의 인자 `name`, `expected` 와 같다.
- 두 번째 인자 `[("Mrs. Claire", "woman"), ("Mr. Jay", "man")]` 는 첫 번째 인자로 넘어가는 테스트 입력 값들이다.
- `("Mrs. Claire", "woman")` 는 `name` 과 `expected` 로 넘어 가게되어, **결과적으로  `name="Mrs. Claire"`, `expected="woman"` 이 테스트 함수의 파라미터로 넘어가게 된다. 그리고 테스트가 진행된다.**
- 다음으로, `("Mr. Jay", "man")` 가 진행되는데, 과정은 위와 똑같이 `name="Mr. Jay"`, `expected="man"` 으로 주입하여 테스트를 하게된다.

- 즉 결과적으로, `test_get_gender(name, expected)` 에서 `[("Mrs. Claire", "woman"), ("Mr. Jay", "man")]` 데이터가 파라미터로 하나씩 넘어가며 테스트가 진행된다.

테스트 함수 내부에는 딱 테스트해야 할 로직만 남아 더 깔끔한 테스트 코드가 되었다.
참고로, 이렇게 입력을 parameter 로 넘기는 테스트를 **parametrized test** 라고 한다.

paramterized test 와 관련된 [이 링크](https://docs.pytest.org/en/stable/example/parametrize.html#parametrizing-test-methods-through-per-class-configuration)를 보면, 다음 내용들을 더 자세히 알 수 있다.

- 각 테스트마다 `id` 를 붙이는 방법 (`id` 를 붙여주면 pytest 실행 시 각 테스트 아이디가 출력되어, 어떤 테스트를 통과했는 지 명시적으로 알 수 있다.
- `pytest.param(data, id)` 를 이용하여 깔끔하게 테스트 데이터를 설정하는 방법



## 2. fixture 가 여러 값을 갖게하기 (Parametrizing fixtures)

fixture 자체가 여러 데이터를 내보내게 할 수 있다.  
다음 예를 보자.

```python
# conftest.py

@pytest.fixture(params=["Mrs. Claire", "Mr. Jay"])  # 여기 주목
def name(request):
    return request.param
```

```python
# test_len_name.py

def test_len_name(name):
    assert len(name) < 10
```

```bash
$ pytest test_len_name.py 
=============================== test session starts ===============================
platform darwin -- Python 3.7.2, pytest-5.4.3, py-1.8.0, pluggy-0.13.1
rootdir: /Users/heumsi/Desktop/dev/playground
plugins: openfiles-0.3.2, arraydiff-0.3, doctestplus-0.3.0, remotedata-0.3.1
collected 2 items                                                                 

test_len_name.py F.                                                         [100%]

==================================== FAILURES =====================================
___________________________ test_len_name[Mrs. Claire] ____________________________

name = 'Mrs. Claire'

    def test_len_name(name):
>       assert len(name) < 10
E       AssertionError: assert 11 < 10
E        +  where 11 = len('Mrs. Claire')

test_len_name.py:48: AssertionError
============================= short test summary info =============================
FAILED test_len_name.py::test_len_name[Mrs. Claire] - AssertionError: assert 11 ...
=========================== 1 failed, 1 passed in 0.10s ===========================
```

`conftest.py` 의 `여기 주목` 을 보면 `@pytest.fixture` 에 `param` 이라는 파라미터가 등장했다.

- `param` 은 `["Mrs. Claire", "Mr. Jay"]` 의 값을 가지는데, 이 fixture 는 순차적으로 이 두 값을 받아 내보낸다.
- 즉, `test_len_name.py` 에서 `test_len_name(name)` 는 `name` 으로 `"Mrs. Claire"` 을 받아 테스트 하고, 그 다음으로 `"Mr. Jay"` 을 받아 테스트 한다.
- `request` 에 대한 자세한 내용은 아래에서 다룬다.

테스트 실행 결과를 보면 `"Mrs. Claire"` 을 입력 받았을 때 실패한 것을 볼 수 있다.



## 3. fixture 에서 데이터 입력받기

fixture 는 우리가 미리 만들어 놓는 데이터 단위라고 볼 수 있다.  
그런데, **테스트 코드로부터 입력을 받아 fixture 를 매번 다르게 만들어야 한다면 어떻게 해야할까?**

fixture 에서도 데이터를 입력받는 방법이 있다.
첫 번째는 `request` 를 이용하는 것이고, 두 번째는 파라미터로 입력 받는 것이다.



### 3.1. `request` fixture 이용하기

일단 다음 예를 보자.

```python
# conftest.py

import pytest

@pytest.fixture
def pretreated_data(request):
    input_data = request.module.input_data
    return [i + 1 for i in input_data]
```

```python
# test_func.py

input_data = [1, 2, 3]

def test_func(pretreated_data):
    assert pretreated_data == [2, 3, 4]
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

테스트가 문제 없이 잘 패스하였다. 무슨 일이 일어난걸까?

- 먼저, fixture 를 테스트 코드와 분리하여 `conftest.py` 에  정의하였다.
- fixture 의 파라미터인 `request` 라는 애가 등장하였다.
    - `request` 는 `FixtureRequest` 라는 객체의 인스턴스인데, pytest 에 의해 자동으로 주입되는 조금 특별한 fixture 다.
    -  `request` 로 테스트 코드 context 에 접근할 수 있다.
    - 위의 경우, `request.module` 로 이 fixture 를 사용하는 테스트 코드의 모듈  `test_func.py` 에 접근하였다.
    - 그리고 `request.module.input_data` 로 이 모듈 내부에 정의된 `input_data` 변수에 접근한다.
- **정리하면 `request` 로 fixture 에서 테스트 코드로의 접근이 가능하다는 것이다.** 
- fixture를 사용하는 테스트 코드의 모듈이 아니라, 함수로 접근하고 싶으면 `request.function` 을 사용하면 된다.
- `request` 에 대한 더 자세한 내용은 [여기](https://docs.pytest.org/en/stable/reference.html?highlight=request#request)서 확인하자.



### 3.2. `@pytest.mark.parametrize` 와 `request.param` 이용하기

> ! 아래 부분은 곧 수정될 예정.



마찬가지로 예를 먼저 보자.

```python
# conftest.py

import pytest

@pytest.fixture
def pretreated_data(request):
    input_data = request.param[0]
    add_number = request.param[1]
    return [i + add_number for i in input_data]
```

```python
# test_func.py

import pytest

data = [
    (1, 4),
    (2, 5),
    (3, 6)
]
add_number = 3

@pytest.mark.parametrize(
    "pretreated_data, expected", [(input_data, add_number)], indirect=["pretreated_data"]
)
def test_func(pretreated_data, expected):
    assert pretreated_data == expected
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

어떤 일이 일어난건지 하나씩 확인해보자.

- `conftest.py` 에서 fixture 를 정의하고 있다.
    - `request.param` 이 등장하는데, 여기에는 이 fixture 로 넘어오는 파라미터 정보를 튜플로 담고있다.
    - `request.param[0]` 과 `request.param[1]` 은 첫 번째 파라미터와 두 번째 파라미터가 담겨있다.
- `test_func.py` 에서는 `@pytest.mark.parametrize` 이 등장하였다.
    - 작성 중 ...







## 4. 참고 및 출처

- https://medium.com/ideas-at-igenius/fixtures-and-parameters-testing-code-with-pytest-d8603abb390a

