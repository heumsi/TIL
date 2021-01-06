# Gunicorn

## 설정

기본적으로 `gunicorn --help` 를 치면 설정할 수 있는 아규먼트들이 쭉 나옴.
아규먼트 방식으로 넘겨도 되지만 다음처럼 별도의 설정 파일에 세팅해놓을 수도 있음.

```python
# gunicorn.py. (파일 이름은 상관없고 확장자가 .py 이어야 함.)

bind = "0.0.0.0:8088"
port = 8088
workers = 2
timeout = 120
worker_class = "sync" 
```

설정 파일로 실행하는 방법은 다음처럼 하면 됨.

```bash
$ gunicorn -c gunicorn.py main:app
```

물론 `main.py` 가 동일 디렉토리에 있어야 함ㅋ



## Worker

크게 Sync, Async 형 워커가 있음.
일반적으로 CPU Bound 는 sync, I/O Bound 는 async 형 워커 쓰면 된다고 보면 됨.

sync 형 워커로는 기본 워커인 `sync` 쓰면 되고,
async 형 워커로는 별도로 `gevent`, `greenlet` 쓰면 된다. 이에 대한 설치 방법은 따로 있으니 검색 바람.


