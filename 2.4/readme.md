
## Подготовка

 Добавляем в ansible-playbook:

    - установку Helm
    - Включение аддона Ingress
    - копирование Helm-чарт в папку пользователя.

## 1. Подготовливаем Helm-чарт для приложения

### 1.1 Упаковываем приложение myapp в Helm-чарт для деплоя в разные окружения.

myapp/
│
├── Chart.yaml
├── values.yaml
└── templates/
    ├── _helpers.tpl
    ├── deployment-frontend.yaml
    ├── deployment-backend.yaml
    ├── service-frontend.yaml
    ├── service-backend.yaml
    ├── ingress.yaml
    └── configmap-frontend.yaml

### 1.2 Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.

В Helm chart каждый файл и папка имеют свою роль для управления и развертывания приложения:

- **Chart.yaml** — основной файл метаданных чарта. Содержит имя чарта, версию, описание и тип. Используется Helm для идентификации и управления версией чарта.

- **values.yaml** — файл с настройками по умолчанию. Здесь задаются параметры, такие как образа контейнеров, количество реплик, порты и другие конфигурации, которые можно переопределять при установке или обновлении чарта.

- **templates/** — папка с шаблонами Kubernetes ресурсов, которые Helm генерирует во время развертывания.

  - **_helpers.tpl** — вспомогательные шаблоны и функции (например, для генерации имен ресурсов). Упрощает повторное использование кода в других шаблонах.

  - **deployment-frontend.yaml** — шаблон Deployment для фронтенд-приложения, описывает развертывание nginx с параметрами из values.yaml.

  - **deployment-backend.yaml** — шаблон Deployment для бэкенда, описывает развертывание контейнера multitool с переменными окружения.

  - **service-frontend.yaml** — шаблон Kubernetes Service для frontend, который обеспечивает сетевой доступ к подам фронтенда.

  - **service-backend.yaml** — шаблон Service для backend, выступающий посредником доступа к backend-подам.

  - **ingress.yaml** — шаблон Ingress, определяет правила маршрутизации HTTP трафика на frontend и backend сервисы по URL путям.

  - **configmap-frontend.yaml** — шаблон ConfigMap, который хранит HTML-файл для фронтенда, содержимое можно менять из values.yaml.

В совокупности эти файлы позволяют максимально гибко и параметризуемо описать развертывание приложения, управляя через values.yaml настройками версий образов, реплик и сетевого доступа. При установке Helm подставляет значения из values.yaml в шаблоны и создаёт готовые Kubernetes объекты.

Это упрощает управление разными версиями, namespace и конфигурациями без дублирования кода или манифестов, позволяя быстро масштабировать и изменять приложение с помощью одной структуры чарта.

### 1.3 Результат выполнения

Запуск Helm-чарт

`helm install my-app .`

Проверяем работу:

`kubectl get pods`

`kubectl get svc`

`kubectl get ingress`

В переменных Helm-чарт изменияем образ приложения для изменения версии.

`help upgrade my-app .\myapp`

Проверяем работу

`curl http://ip`

## 2. Запускаем две версии в разных неймспейсах

### 2.1 Создаём namespaces

`kubectl create namespace app1`

`kubectl create namespace app2`


### 2.2 Одну версию приложения запускаем в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.

```
helm install frontend-v1 ./myapp \
  --namespace app1 \
  --set frontend.image.tag="1.27" \
  --set backend.image.tag="latest"
```

```
helm install frontend-v2 ./myapp \
  --namespace app1 \
  --set frontend.image.tag="1.26" \
  --set backend.image.tag="latest"
```

```
helm install frontend-v3 ./myapp \
  --namespace app2 \
  --set frontend.image.tag="latest" \
  --set backend.image.tag="latest"
```

### 2.3 Результат

Посмотреть список всех релизов можно командой

`helm list -A`

Просмотреть поды

`kubectl get pods -n app1`

`kubectl get pods -n app2`
