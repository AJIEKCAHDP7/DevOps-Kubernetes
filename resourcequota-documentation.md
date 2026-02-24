Что такое ResourceQuota?

ResourceQuota — это объект Kubernetes, который ограничивает общее потребление ресурсов в namespace.

Если лимит превышен → новые ресурсы создать нельзя ❌

🔹 Чем отличается от LimitRange?
LimitRange	ResourceQuota
Ограничивает каждый Pod	Ограничивает весь namespace
min / max / default	общий суммарный лимит
Работает на уровне контейнера	Работает на уровне namespace
🔹 Пример ResourceQuota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
  namespace: dev
spec:
  hard:
    pods: "10"                   # максимум 10 Pod
    requests.cpu: "4"            # всего 4 CPU requests
    requests.memory: 8Gi         # всего 8Gi requests
    limits.cpu: "6"              # максимум 6 CPU limits
    limits.memory: 12Gi          # максимум 12Gi memory limits

🔹 Что это значит?

В namespace dev:

нельзя создать больше 10 Pod

суммарные CPU requests не больше 4

суммарная память не больше 8Gi

Если попытаться создать 11 Pod:

Error: exceeded quota

🔹 Проверка квоты
kubectl describe resourcequota -n dev


Покажет:

used

hard limit

🔹 Ограничение количества объектов

Можно ограничивать:

spec:
  hard:
    pods: "10"
    services: "5"
    configmaps: "20"
    secrets: "20"
    persistentvolumeclaims: "5"

🔹 Ограничение PVC storage
spec:
  hard:
    requests.storage: 50Gi


Это значит:
→ суммарный размер PVC не больше 50Gi

🔹 Важно понимать

ResourceQuota работает только если:

у Pod указаны requests/limits

иначе quota может не применяться корректно

🔥 Что будет если нет requests?

Если включена quota по requests.cpu,
а Pod не указал requests → он не создастся.

🔥 Типичный продакшен namespace

Обычно:

LimitRange → чтобы задать default

ResourceQuota → чтобы ограничить суммарное потребление
