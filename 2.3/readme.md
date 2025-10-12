## 1. ConfigMaps

### 1.1 Создаём Deployment с двумя контейнерами nginx и multitool

deployment.yaml

Данный манифест Kubernetes описывает деплойменты и службы для фронтенда и бэкенда, а также ингресс для маршрутизации трафика следующим образом:

- Деплоймент фронтенда запускает контейнер на базе Nginx, который слушает порт 80.
- Деплоймент бэкенда запускает контейнер multitool, слушающий порт 8080.
- Соответствующие службы открывают доступ к фронтенду на порту 80 и к бэкенду на порту 8080.
- Ингресс маршрутизирует путь '/' на фронтенд, а '/api' на бэкенд, с правилом перезаписи пути на '/'.

### Подробности по компонентам

**Деплоймент фронтенда**
- 1 реплика образа `nginx`.
- Контейнер слушает порт 80.
- Метка для поиска: `app: frontend`.

**Деплоймент бэкенда**
- 1 реплика образа `wbitt/network-multitool` с переменной окружения `HTTP_PORT=8080`.
- Контейнер слушает порт 8080.
- Метка для поиска: `app: backend`.

**Службы**
- `frontend-service` выбирает поды с меткой `app: frontend`, обслуживает порт 80.
- `backend-service` выбирает поды с меткой `app: backend`, обслуживает порт 8080.

**Ингресс**
- Аннотация для nginx-ingress контроллера: `nginx.ingress.kubernetes.io/rewrite-target: /`.
- Маршрутизация пути '/' с префиксным совпадением на `frontend-service` порт 80.
- Маршрутизация пути '/api' с префиксным совпадением на `backend-service` порт 8080.

Это типичный пример разделения фронтенда и бэкенда в Kubernetes с использованием ингресс-контроллера для маршрутизации http-запросов.

Не забываем включить аддон ingress.

```
microk8s enable ingress
```

### 1.2 Подключаем веб-страницу через ConfigMap

configmap-web.yaml

Это пример Kubernetes ConfigMap с именем `frontend-webpage`, который содержит HTML страницу в данных под ключом `index.html`.

### Описание ConfigMap:
- **apiVersion:** v1 — версия API Kubernetes.
- **kind:** ConfigMap — тип ресурса.
- **metadata:** включает имя ConfigMap — `frontend-webpage`.
- **data:** содержит ключ `index.html` с HTML-разметкой веб-страницы, которая выводит заголовок "Тестовая страница" и приветственное сообщение на русском языке.

Эта ConfigMap может быть использована в подах Kubernetes для монтирования веб-страницы как файла или передаче конфигурации в контейнер. Например, монтирование `index.html` в веб-сервер, чтобы отобразить эту страницу.

### 1.3 Проверяем доступность

`curl -k http://158.160.112.55`

### 1.4 Шаги выполнения

`microk8s enable ingress`

`microk8s kubectl apply -f deployment.yaml`

`microk8s kubectl apply -f configmap-web.yaml`

## 2. Настройка HTTPS с Secrets

### 2.1 Модифицируем предыдущий Deployment с двумя контейнерами nginx и multitool

deployment-ingress-tls.yaml

Предоставленные манифесты Kubernetes описывают простое приложение с фронтенд и бэкенд компонентами, которые доступны через сервисы и ингресс ресурс.

### Что изменилось

- **Ингресс**
  - Обрабатывает запросы на домен `myapp.example.com` с использованием TLS и секрета `myapp-tls`.

### 2.2 Генерируем сертификат (выполняется в терминале)

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
```

### 2.3 Создаём Secret

secret-tls.yaml

Есть два варианта создания:

- команда

`microk8s kubectl create secret tls myapp-tls --cert=tls.crt --key=tls.key`

- команда

`microk8s kubectl get secret myapp-tls -o yaml > secret-tls.yaml`

Предоставленный манифест Kubernetes описывает объект Secret типа `kubernetes.io/tls`. Этот секрет называется `myapp-tls` и находится в пространстве имён `default`. Он содержит два элемента данных в закодированном виде base64:

- `tls.crt`: TLS-сертификат
- `tls.key`: TLS-ключ

Этот секрет можно использовать для обеспечения TLS-данными приложений в Kubernetes, например, для настройки входящего трафика через ingress или для приложений, работающих по протоколу TLS.

### 2.4 Проверяем доступность

`curl -k https://158.160.112.55`

### 2.5 Шаги выполнения

`microk8s enable ingress`

`microk8s kubectl apply -f deployment-ingress-tls.yaml`

`microk8s kubectl apply -f configmap-web.yaml`

`microk8s kubectl create secret tls myapp-tls --cert=tls.crt --key=tls.key`

### 3.2 Создаём Role (только просмотр логов и описания подов) и RoleBinding

role-pod-reader.yaml

Данный объект Role в Kubernetes задает роль с именем "pod-viewer" в пространстве имен "default". 

Эта роль предоставляет права только на чтение (verbs: "get", "list", "watch") ресурсов "pods" (поды) и "pods/log" (логи подов) в указанном неймспейсе. То есть пользователи или сервисные аккаунты, которым привязана эта роль, могут просматривать информацию о подах и их логах, но не могут их изменять или удалять.

rolebinding-developer.yaml

Данный объект RoleBinding с именем "developer-pod-viewer-binding" в пространстве имен "default" связывает пользователя с именем "developer" с ролью "pod-viewer".

