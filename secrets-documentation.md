Secret — это объект Kubernetes, предназначенный для хранения конфиденциальных данных, таких как пароли, токены, API-ключи или TLS-сертификаты. Он позволяет хранить чувствительные данные отдельно от Pod и контейнерного образа, чтобы не включать их в код приложения.

Типичные данные в Secret:
пароли баз данных
API-ключи
токены
TLS сертификаты
SSH ключи
Docker registry credentials
Secrets похожи на ConfigMap, но предназначены именно для секретных данных.

Чтобы не кодировать данные вручную, можно использовать stringData.

Kubernetes автоматически преобразует значения в base64 и сохраняет их в data.

Основные типы Secrets:
1. Opaque (самый распространенный). Используется для хранения произвольных данных.
2. TLS Secret. Используется для HTTPS сертификатов.
3. Docker Registry Secret. Для доступа к private registry.
4. Basic Auth Secret. Для хранения username/password.
5. SSH Secret. Для SSH ключей.

Как прочитать Secret

Secrets хранятся в base64.

Получить значение:

kubectl get secret db-credentials -o jsonpath='{.data.username}' | base64 --decode

Получить весь Secret:

kubectl get secret db-credentials -o yaml
