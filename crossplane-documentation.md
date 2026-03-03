Crossplane — это расширение Kubernetes, которое превращает его в универсальную control plane для инфраструктуры.
Идея простая: вы управляете не только Pod’ами и Service’ами, но и облачными ресурсами (VPC, базы, кластеры, S3, DNS и т.д.) — прямо через Kubernetes API.

Проще говоря:
👉 Kubernetes управляет Kubernetes
👉 Crossplane позволяет Kubernetes управлять облаком

Что делает Crossplane
Он добавляет в Kubernetes новые CRD (Custom Resources), например:
RDSInstance
S3Bucket
VPC
IAMRole
KubernetesCluster

И когда вы создаёте такой объект в YAML — Crossplane создаёт реальный ресурс в облаке.

Как это выглядит
apiVersion: rds.aws.crossplane.io/v1beta1
kind: DBInstance
spec:
  forProvider:
    region: eu-central-1
    dbInstanceClass: db.t3.medium

После применения:
В AWS создаётся реальная RDS база
Crossplane следит за её состоянием
Если ресурс удалён вручную — он может восстановить его

Архитектура
Kubernetes API
     |
Crossplane Core
     |
Provider (AWS/Azure/GCP/etc)
     |
Cloud API

Компоненты:
Crossplane Core
Providers
Managed Resources
Compositions
Providers

Crossplane сам ничего не создаёт — он использует провайдеры.

Популярные:

AWS
Azure
GCP
Kubernetes
Helm
GitHub
Cloudflare

Provider — это контроллер, который знает, как общаться с API конкретной системы.

Главное преимущество — Compositions

Это самая сильная часть Crossplane.

Вы можете описать платформенный абстрактный ресурс, например:

kind: MyCompanyDatabase

А внутри Composition будет:

VPC

Subnet

Security Group

RDS

Secrets

IAM

Разработчик создаёт:

kind: MyCompanyDatabase

А Crossplane разворачивает всю инфраструктуру.

Это называется Platform Engineering.

Crossplane vs Terraform
Terraform

CLI инструмент

вне Kubernetes

state хранится отдельно

Crossplane

работает внутри Kubernetes

использует Kubernetes API

декларативный reconciliation loop

GitOps-friendly

Главное отличие:

Terraform → “запустил и вышел”
Crossplane → “постоянный контроллер”

Где используется

Crossplane хорошо подходит:

для Internal Developer Platform (IDP)

для self-service инфраструктуры

для multi-cloud

для GitOps (ArgoCD/Flux)

Пример реального сценария

Компания хочет:

Разработчики не должны:

знать AWS

создавать IAM

настраивать VPC

Они просто пишут:

kind: Application

Crossplane:

создаёт кластер

создаёт базу

создаёт S3

настраивает сеть

Crossplane на bare metal

Можно использовать для:

создания облачных ресурсов из on-prem кластера

управления другими Kubernetes кластерами

гибридной инфраструктуры

Пример:

On-prem Kubernetes управляет:

AWS RDS

GCP Bucket

Azure DNS



1️⃣ Установка Crossplane
Через Helm (рекомендуется)
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

helm upgrade --install crossplane crossplane-stable/crossplane \
  --namespace crossplane-system \
  --create-namespace

Проверка:

kubectl get pods -n crossplane-system

Должны быть запущены:

crossplane

crossplane-rbac-manager

2️⃣ Установка Provider (пример: AWS)

Crossplane сам ресурсы не создаёт — нужен Provider.

Установка AWS provider
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-aws
spec:
  package: xpkg.upbound.io/crossplane-contrib/provider-aws:v0.54.2

Применяем:

kubectl apply -f provider-aws.yaml

Проверяем:

kubectl get providers
kubectl get pods -n crossplane-system

Появятся новые pod'ы provider-aws.

3️⃣ Доступ к облаку (credentials)

Crossplane работает через Kubernetes Secret.

Пример для AWS

Создаём файл creds.conf:

[default]
aws_access_key_id=AKIA...
aws_secret_access_key=SECRET...

Создаём Secret:

kubectl create secret generic aws-creds \
  -n crossplane-system \
  --from-file=creds=./creds.conf
Создаём ProviderConfig
apiVersion: aws.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: aws-default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-creds
      key: creds
kubectl apply -f providerconfig.yaml
4️⃣ Создание ресурса (пример: S3 Bucket)
apiVersion: s3.aws.crossplane.io/v1beta1
kind: Bucket
metadata:
  name: my-crossplane-bucket
spec:
  forProvider:
    region: eu-central-1
  providerConfigRef:
    name: aws-default

Применяем:

kubectl apply -f bucket.yaml

Проверяем:

kubectl get buckets
kubectl describe bucket my-crossplane-bucket

Если всё ок — бакет появится в AWS.

Что теперь работает

Теперь Kubernetes:

создаёт облачные ресурсы

следит за их состоянием

восстанавливает drift

удаляет ресурс при удалении CR

Это полноценный reconciliation loop.

Production рекомендации
1️⃣ Отдельный management cluster

Лучше ставить Crossplane в отдельный кластер.

2️⃣ Ограниченные IAM права

Создавайте отдельный IAM user/role с минимальными правами.

3️⃣ GitOps

Лучше применять CR через:

ArgoCD

Flux

4️⃣ Использовать Compositions

В production редко создают raw Bucket или RDS.
Обычно делают абстракции:

kind: CompanyDatabase
Если у вас bare metal

Очень частый сценарий:

On-prem Kubernetes
→ Crossplane
→ AWS RDS
→ AWS S3
→ Cloudflare DNS

Это гибридная модель.

Частые проблемы

❌ Provider не Ready → проблема с образом или версией

❌ Resource stuck в Creating → проблемы с IAM

❌ Credentials error → неправильно создан Secret

❌ API version mismatch → версия provider не совпадает
