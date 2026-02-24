Horizontal Pod Autoscaler (HPA) — один из ключевых механизмов Kubernetes.

🔹 Что такое HPA?

Horizontal Pod Autoscaler автоматически:

увеличивает количество Pod'ов 📈

уменьшает количество Pod'ов 📉

в зависимости от нагрузки.

🔹 Как работает HPA?

HPA смотрит на:

CPU usage (чаще всего)

Memory usage

Custom metrics (Prometheus)

External metrics

И изменяет replicas в Deployment / StatefulSet.

🔹 Архитектура
Pod → Metrics Server → HPA → Deployment → ReplicaSet → Pods


⚠ HPA требует установленный metrics-server

Проверка:

kubectl get deployment metrics-server -n kube-system

🔹 Пример Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: app
          image: nginx
          resources:
            requests:
              cpu: 100m
            limits:
              cpu: 500m


⚠ HPA работает только если есть requests

🔹 Пример HPA (CPU-based)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app          # что масштабировать
  minReplicas: 2           # минимум Pod
  maxReplicas: 10          # максимум Pod
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70   # масштабировать если >70%

🔹 Что означает averageUtilization: 70?

Если средний CPU usage > 70% от requests → увеличить Pod.

Пример:

requests: 100m

Pod использует 80m

80% > 70% → HPA добавит Pod

🔹 Проверка HPA
kubectl get hpa
kubectl describe hpa web-app-hpa


Показывает:

Current CPU

Target

Replicas

🔹 Как протестировать

Создать нагрузку:

kubectl run -i --tty load-generator --rm --image=busybox -- sh


Внутри:

while true; do wget -q -O- http://web-app; done


Потом:

kubectl get hpa -w

🔹 Разница HPA / VPA / Cluster Autoscaler
Тип	Что масштабирует
HPA	Количество Pod
VPA	CPU/Memory Pod
Cluster Autoscaler	Количество Node
