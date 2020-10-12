`Dockerfile` 에서 poetry 어떻게 쓸지 이래저래 논의가 많은데, 개인적으로 아래 샘플이 제일 깔끔한거 같다.

```dockerfile
FROM python:3.8.5-slim

RUN python -m pip install --upgrade pip && \
    pip install poetry==1.0.10 && \
    POETRY_VIRTUALENVS_CREATE=false poetry install --no-dev --no-root
```

- `pip` 로 poetry 를 설치한다.
- `POETRY_VIRTUALENVS_CREATE=false` 옵션으로, 가상환경을 안 만들고 바로 패키지를 설치한다. (도커로 띄워주기 때문에 굳이 가상환경할 필요가 없다)
- `--no-dev` 옵션으로 배포용 패키지만 설치
- `--no-root` 옵션으로 의존성을 가진 패키지만 설치 (즉 현재 이 poetry를 사용 중인 파이썬 패키지, 즉 내가 만든 패키지는 설치하지 않음.)