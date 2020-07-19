```python
import asyncio
import threading
from concurrent.futures import ThreadPoolExecutor
from urllib.request import urlopen  # Blocking I/O

urls = [
    "http://daum.net",
    "https://naver.com",
    "http://tistory.com",
]


async def fetch(url, executor):
    thread_name = threading.current_thread().getName()
    print(f"thread name: {thread_name}, url: {url}")
    
    response = await loop.run_in_executor(executor, urlopen, url)
    return response.read()[0:5]  # 일부만 가져옴


async def main():
    # 쓰레드 풀 생성
    executor = ThreadPoolExecutor(max_workers=10)

    # asyncio.Task 객체 리스트를 생성
    futures = [asyncio.ensure_future(fetch(url, executor)) for url in urls]

    # 결과 취합
    result = await asyncio.gather(*futures)

    print(f"result : {result}")


if __name__ == "__main__":
    # 루프 초기화
    loop = asyncio.get_event_loop()

    # 작업 완료까지 대기
    loop.run_until_complete(main())
    loop.close()
```

- `async` 와 `asyncio` 문법은 어느정도 패턴화 되어 있는거 같음.
    - 그냥 그대로 답습하는게 맞을 듯...
- 추가 설명은  https://dojang.io/mod/page/view.php?id=2469  참고.

