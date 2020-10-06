다음과 같이 Pod이 `Terminating` 상태에 계속 Stuck 되어있을 때, 다음 명령어로 바로 kill 할 수 있다.

```bash
$ kubectl delete pod xxx --now
# 또는 
$ kubectl delete pod foo --grace-period=0 --force
```

예를 들면 다음과 같은 상황에서.

```bash
$ kubectl get all -n airflow
NAME                   READY   STATUS        RESTARTS   AGE
pod/airflow-worker-0   3/3     Terminating   0          16m
pod/airflow-worker-1   3/3     Terminating   0          16m
```

아래 명령어를 입력해준다.

```bash
$ kubectl delete pod/airflow-worker-0 --grace-period=0 --force -n airflow
```

