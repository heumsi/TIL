```10
python -m grpc_tools.protoc --proto_path=./ --python_out=../app --grpc_python_out=../app health_probe.proto
```

---

위와 같이 하면 만들어지는 `_pb2_grpc.py` 에  `import` 가 다음과 같이 상대 경로로 되어있음.

```python
# health_probe_pb2_grpc.py

import health_probe_pb2
...
```

상대 경로로 되어있기 때문에 프로젝트 단위의 run 을 할 때, **정상적인 실행이 안됨** 
말할 것도 없이 찾을 수 없는 패키지라며 `import ` 에러가 뜬다.

[이에 관한 깃허브 이슈](https://github.com/protocolbuffers/protobuf/issues/1491)가 아주 활발히 있었는데. 딱 결론만 말하면 다음임.

```bash
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. app/proto/health_probe.proto
```

처음에 이게 뭔가 했는데, 가장 마지막 인자 `app/proto/health_probe.proto` 의 경로를 기준 잡아 `--out` 경로를 잡은 뒤 `.`  에 출력하는거 같다.