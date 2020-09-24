# 오토 스케일링 HPA

- 포드 스케일링의 두 가지 방법
    - HPA(Horizontal Pod Autoscaler): 포드 자체를 복제하여, 처리할 수 있는 포드의 개수를 늘리는 방법
    - VPA: 리소스를 증가시켜 포드의 사용 가능한 리소스를 늘리는 방법
    - CA: 번외로 클러스터 자체를 늘리는 방법 (노드 추가)
- HPA
    - 쿠버네티스에는 기본 오토스케일링 기능 내장
    - CPU 사용률을 모니터링하여 실행된 포드의 개수를 늘리거나 줄임
- VPA, CA
    - 공식 쿠버네티스에서 제공하지는 않으나, 클라우드 서비스에서 제공

다음 명령어로 기존 Pod에 HPA 가능

```bash
$ kubectl autoscale deployment my-app --max 6 --min 4 --cpu-percent 50
```

다음과 같이 `.yaml` 작성도 가능

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: php-apache
  targetCPUUtilizationPercentage: 50  # cpu 사용율이 50% 넘어가면 scale out 함.
```

다음과 같이 확인 가능

```bash
$ kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          16s
```

