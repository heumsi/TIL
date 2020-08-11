### 관련 아티클

https://spoqa.github.io/2019/02/15/python-timezone.html  
늘 스포카에게 감사하고 있다. 사랑해요 스포카.

<br>

### naive datetime vs aware datetime

- naive datetime

    - 타임존이 명시가 안된 `datetime` 객체
    - ex. `datetime.datetime(2020, 8, 5, 16, 45, 12, 184060)`

    - 쓰지 말자. 헷갈린다. 그리고 aware datetime 하고도 연산도 안된다.

- aware datetime

    - 타임존이 명시가 된 `datetime` 객체
    - ex. `datetime.datetime(2020, 8, 5, 16, 43, 9, 417500, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)`
    - 이거 쓰자. **앞으로 `datetime` 객체는 반드시 타임 존을 명시해서 만들자.**

<br>

### 특정 timezone 에 맞게 얻어오기

- `datetime` 메서드인 `.astimezone()`  을 이용한다.

<br>

### 예제

timezone 설정은 써드파티인 pytz 이용한다.

```python
>>> import datetime
>>> from pytz import utc, timezone

# aware datetime 객체 생성 
>>> utc_now = datetime.datetime.now(utc)
>>> utc_now
datetime.datetime(2020, 8, 5, 8, 6, 57, 144657, tzinfo=<UTC>)

>>> kst_now = datetime.datetime.now(timezone('Asia/Seoul')) 
>>> kst_now
datetime.datetime(2020, 8, 5, 17, 7, 56, 127873, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

# 특정 timezone의 aware datetime 얻어오기
>>> utc_now.astimezone(timezone('Asia/Seoul'))
datetime.datetime(2020, 8, 5, 17, 12, 1, 390826, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>)

>>> kst_now.astimezone(utc)
datetime.datetime(2020, 8, 5, 8, 12, 10, 292835, tzinfo=<UTC>)
```

<br>

### datetime + pytz.timezone 문제

``datetime.datetime.now(timezone('Asia/Seoul'))` 는 잘 되는데, 문제는 다음과 같이 내가 특정 날짜를 지정해줄 때다.

```python
>>> datetime.datetime(2020, 12, 31, 0, 0, 0, tzinfo=timezone('Asia/Seoul'))
datetime.datetime(2020, 12, 31, 0, 0, tzinfo=<DstTzInfo 'Asia/Seoul' LMT+8:28:00 STD>)
```

기대했던  `KST+9:00:00 STD` 와 달리 `LMT+8:28:00 STD` 이라는 값이 등장한다 ;;;

실제로 문제가 일어나는 구간은 `timestamp` 로 변환한 뒤, 다시 `datetime` 으로 받아올 때다.

```python
>>> dt_kst = datetime.datetime(2020, 12, 31, 0, 0, 0, tzinfo=timezone('Asia/Seoul'))  # KST 기준의 2020-12-31 00:00:00
>>> ts = dt_kst.timestamp()  # UTC 기준의 timestamp 로 변환한다.

>>> datetime.datetime.fromtimestamp(ts, tz=timezone('Asia/Seoul'))
datetime.datetime(2020, 12, 31, 0, 32, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>) 
```

마지막 결과 보면 알겠지만... `2020-12-31 00:00:00` 이 아니라 `2020-12-31 00:32:00` 라는 값이 나온다.

`pytz.timezone` 와 `datetime` 을 같이 쓰면 안되는 이유도 스포카 블로그에 잘 나와있음...

<br>

### 결론 및 사용 가이드

`pytz.timezone` 을 안쓰고, 표준 모듈인 `datetime.timezone` 을 쓴다.

```python
import datetime

# datetime.timezone 으로 timezone 을 정의하자.
KST = datetime.timezone(datetime.timedelta(hours=9))
UTC = datetime.timezone(datetime.timedelta(hours=0))

# datetime 객체를 만들 때는 항상 timezone 을 주입해주자.
kst_now = datetime.datetime.now(KST) 

# timestamp 에서 KST 시간대의 datetime 객체를 만드는 방법은 다음과 같다.
datetime.datetime.fromtimestamp(1596330000, tz=KST)

# naive datetime 에 timezone 을 주입해주는 방법은 다음과 같다.
# 내가 만든 datetime 이 아닌 경우, (이미 naive 로 만들어 진 경우) 타임 존만 주입해줘야 할 때 유용하다.
datetime.datetime.now().replace(tzinfo=KST)

# aware datetime 을 특정 타임존에 맞게 가져오는 방법은 다음과 같다.
datetime.datetime.now(UTC).astimezone(KST)
```
