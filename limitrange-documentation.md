LimitRange — это механизм в Kubernetes, который ограничивает ресурсы контейнеров в namespace.

🔹 Что такое LimitRange?

LimitRange:

задаёт минимальные и максимальные CPU/Memory

может задать значения по умолчанию

работает на уровне namespace

Если Pod не соответствует правилам → он не создастся ❌

🔹 Зачем нужен?

Чтобы:

никто не запустил Pod с 10 CPU в маленьком кластере

задать default limits

контролировать ресурсы в namespace

🔹 Что происходит после применения?
kubectl apply -f limitrange-example.yaml


Теперь:

Pod без ресурсов → получит default

Pod с 2 CPU → будет отклонён

Pod с 50m CPU → будет отклонён

🔹 Проверить
kubectl describe limitrange

🔹 Важные типы
type	Что ограничивает
Container	Отдельные контейнеры
Pod	Весь Pod
PersistentVolumeClaim	Размер PVC
🔹 Пример ограничения PVC
apiVersion: v1
kind: LimitRange
metadata:
  name: pvc-limit
spec:
  limits:
    - type: PersistentVolumeClaim
      min:
        storage: 1Gi
      max:
        storage: 10Gi

🔥 Часто путают
LimitRange	ResourceQuota
Ограничивает каждый Pod	Ограничивает весь namespace
min/max/default	общий лимит ресурсов
