## 로컬 모듈을 import 못해오는 이슈

나 말고도 이미 스택 오버플로에 많은 이슈가 있었음...

- https://stackoverflow.com/questions/47998552/apache-airflow-dag-cannot-import-local-module/54100781
    - 결론 : 아직 해결 안됬다고 함
- https://stackoverflow.com/questions/50150384/importing-local-module-python-script-in-airflow-dag
    - `export AIRFLOW_HOME=path_to_my_airflow_home_dir` 이걸 해보라고 함.
- https://github.com/puckel/docker-airflow/issues/101
    - 음.. 느낌이 오건데. `AIRFLOW_HOME = /root/airflow` 이 ROOT 디렉토리고 여기서부터 `/dags` 등으로 이루어져있어야 될 거 같음.
    - 근데 지금 우리는 `AIRFLOW_HOME` 밑에 `/root/airflow/git/repo` 에 `/dags` 및 다른게 있거든....
    - PYTHONPATH 는 싹 무시하고 얘가 그냥 오버라이트 하는거 같은데 말이지... 

이게 뭔일이람 ...

