# Pytest tutorial

<br>

## 1. 설치

```bash
$ pip install pytest
```

```python
>>> import pytest
>>> pytest.__version__
'5.4.3'
```

글 쓰는 시점 기준, `5.4.3` 버전 사용.

<br>

## 2. 기본 사용법

### 테스트 코드 작성

먼저 테스트 대상이 되는 함수 작성하자.

```python
# func.py

def func():
    return "hello pytest"
```

다음 테스트 코드로 위 함수를 테스트 하자.

```python
# test_func.py

from func import func

def test_func():
    assert func() == "hello pytest"
```

- **함수 이름에 prefix 로 `test_` 를 붙여주면 pytest 가 인식하여 작동한다.**
- 파이썬 내장 패키지인 `unittest ` 보다 **훨씬 간결**한 코드. 
- 이미 `unittest` 로 쓰인 코드도 pytest 로 작동 가능하다.
- 위에서는 함수 단위로 만들었지만, 다음과 같이 클래스 단위로 작성 가능 (메서드 이름이 `test_` 로 시작해야 함)

```python
class TestFunc:
    def test_func():
        assert func() == "hello pytest" 
```

<br>

### pytest 실행

```bash
$ pytest test_something.py

# 실행 결과
===== test session starts =====
test_something.py .       [100%]

====== 1 passed in 0.01s =======
```

- `.` 는 패스. `F` 은 실패. `E` 는 예외로 표시 된다.

<br>

## 3. 예외 테스트

예외(Exception) 을 잘 raise 하는지는 다음과 같이 `pytest.raises` 로 확인해볼 수 있다.

```python
# func.py

def func_raise():
    raise Exception("error message")
```

```python
# test_func_raise.py

from pytest import raises
from func import func_raise

def test_func_raise():
    expected_exception = Exception
    expected_msg = "error mes"  # Exception message 의 일부만 넣어도 된다.
    with raises(expected_exception, match=expected_msg):
        func_raise()
```

코드 작성 후, pytest 를 돌려보면 다음과 같이 테스트에 통과했다고 나온다.

```bash
$ pytest test_func_raise.py

# 실행 결과
===== test session starts =====
test_func_raise.py .       [100%]

====== 1 passed in 0.01s =======
```

<br>

## 4. assert 주의 사항

다음 코드를 작성해보자.

```python
# func.py

import numpy as np

def func_equal():
    return np.array([1, 2, 3, 4, 5])
```

```python
# test_func_equal.py

import numpy as np
from func import func_equal

def test_func_equal():
    assert func_equal() == np.array([1, 2, 3, 4, 5])
```

pytest 를 돌려보면 다음과 같은 결과가 나온다.

```bash
$ pytest test_func_equal.py

===== test session starts =====                                                                                                                                                            test_func_equal.py F      [100%]

===== FAILURES =====
____ test_func_equal ____
    def test_func_equal():
>       assert func_equal() == np.array([1, 2, 3, 4, 5])
E       ValueError: The truth value of an array with more than one element is ambiguous. Use a.any() or a.all()

test_func_equal.py:7: ValueError
```

결과가 `fail` 로 뜨는 이유는, `func_equal() == np.array([1, 2, 3, 4, 5])` 의 결과가 `True` 또는 `False`, 즉 `bool` 타입이 아니기 때문이다.

`==` 연산자는 객체의 `__eq__` 를 호출하는데, `numpy.array` 객체의 `__eq__` 은 `bool` 타입을 반환하지 않는다.  
 `func_equal() == np.array([1, 2, 3, 4, 5])` 는 `[True, True, True, True, True]` 를 반환한다.  
`bool` 타입이 아니므로, `ValueError` 를 뱉고 있는 것이다.

**따라서, `assert` 안에 `==` 로 두 객체를 비교 시, `__eq__` 가 `bool` 타입을 반환하는지 고려해야 한다.**

<br>

> `numpy.array` 비교 관련 이슈가 깃허브에 이미 올라와 있다.  
> [여기](https://github.com/pytest-dev/pytest/issues/5347)를 클릭하면 확인 할 수 있다. (아직까지 깔끔하게 해결되진 않은 듯)
>
> 나같은 경우 `numpy.array.tolist()` 메서드로 `list` 변환한 뒤 비교 함...

