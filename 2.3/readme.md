## 1. ConfigMaps

### 1.1 Создаём Deployment с двумя контейнерами nginx и multitool

https://github.com/DioRoman/kubernetes/blob/main/2.3/deployment.yaml

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

https://github.com/DioRoman/kubernetes/blob/main/2.3/configmap-web.yaml

Это пример Kubernetes ConfigMap с именем `frontend-webpage`, который содержит HTML страницу в данных под ключом `index.html`.

### Описание ConfigMap:
- **apiVersion:** v1 — версия API Kubernetes.
- **kind:** ConfigMap — тип ресурса.
- **metadata:** включает имя ConfigMap — `frontend-webpage`.
- **data:** содержит ключ `index.html` с HTML-разметкой веб-страницы, которая выводит заголовок "Тестовая страница" и приветственное сообщение на русском языке.

Эта ConfigMap может быть использована в подах Kubernetes для монтирования веб-страницы как файла или передаче конфигурации в контейнер. Например, монтирование `index.html` в веб-сервер, чтобы отобразить эту страницу.

### 1.3 Шаги выполнения

`microk8s enable ingress`

<img width="849" height="369" alt="Снимок экрана 2025-10-13 223236" src="https://github.com/user-attachments/assets/28803740-c412-4187-ab58-b9eab8b049b2" />

`microk8s kubectl apply -f deployment.yaml`

`microk8s kubectl apply -f configmap-web.yaml`

<img width="674" height="184" alt="Снимок экрана 2025-10-13 223248" src="https://github.com/user-attachments/assets/94890f7f-e103-4669-860c-c3ecd8462170" />

### 1.4 Проверяем доступность

`curl -k http://158.160.112.55`

<img width="674" height="118" alt="Снимок экрана 2025-10-13 210353" src="https://github.com/user-attachments/assets/993f3732-740f-4e51-88d9-45249a924bf5" />

## 2. Настройка HTTPS с Secrets

### 2.1 Модифицируем предыдущий Deployment с двумя контейнерами nginx и multitool

https://github.com/DioRoman/kubernetes/blob/main/2.3/deployment-ingress-tls.yaml

Предоставленные манифесты Kubernetes описывают простое приложение с фронтенд и бэкенд компонентами, которые доступны через сервисы и ингресс ресурс.

### Что изменилось

- **Ингресс**
  - Обрабатывает запросы на домен `myapp.example.com` с использованием TLS и секрета `myapp-tls`.

### 2.2 Генерируем сертификат (выполняется в терминале)

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
```

<img width="953" height="462" alt="Снимок экрана 2025-10-13 223931" src="https://github.com/user-attachments/assets/a8877075-c2b4-4957-ba6c-03f29c8196fa" />

### 2.3 Создаём Secret

https://github.com/DioRoman/kubernetes/blob/main/2.3/secret-tls.yaml

Есть два варианта создания:

- команда

`microk8s kubectl create secret tls myapp-tls --cert=tls.crt --key=tls.key`

- команда

`microk8s kubectl get secret myapp-tls -o yaml > secret-tls.yaml`

Предоставленный манифест Kubernetes описывает объект Secret типа `kubernetes.io/tls`. Этот секрет называется `myapp-tls` и находится в пространстве имён `default`. Он содержит два элемента данных в закодированном виде base64:

- `tls.crt`: TLS-сертификат
- `tls.key`: TLS-ключ

Этот секрет можно использовать для обеспечения TLS-данными приложений в Kubernetes, например, для настройки входящего трафика через ingress или для приложений, работающих по протоколу TLS.

### 2.4 Шаги выполнения

`microk8s enable ingress`

`microk8s kubectl apply -f deployment-ingress-tls.yaml`

`microk8s kubectl apply -f configmap-web.yaml`

`microk8s kubectl create secret tls myapp-tls --cert=tls.crt --key=tls.key`

<img width="1023" height="56" alt="Снимок экрана 2025-10-13 224137" src="https://github.com/user-attachments/assets/2372b397-05d2-4e6d-adea-793fcf27dd07" />

### 2.5 Проверяем доступность

`curl -k https://158.160.112.55`

<img width="717" height="125" alt="Снимок экрана 2025-10-12 185838" src="https://github.com/user-attachments/assets/af87323e-52b9-4139-9f9d-4d0d57e3de93" />

## 3. Настройка RBAC

### 3.1 Включаем RBAC в microk8s

`microk8s enable rbac`

<img width="453" height="149" alt="Снимок экрана 2025-10-13 211929" src="https://github.com/user-attachments/assets/53ef00cb-6471-4485-8ecb-e94da86dd780" />

RBAC (Role-Based Access Control) в MicroK8s используется для управления доступом пользователей и сервисов к ресурсам Kubernetes кластера, разделяя права на основе ролей. В MicroK8s RBAC можно включить через аддон с помощью команды microk8s.enable rbac. После включения RBAC создаются роли (Role или ClusterRole), которые определяют набор разрешений (что пользователь или сервис может делать с ресурсами), и связываются с пользователями или группами через RoleBinding или ClusterRoleBinding.

### 3.2 Создаём сертификаты для пользователя

Команды выполняют последовательные шаги для создания ключа, запроса на подпись сертификата (CSR) и подписи этого запроса существующим CA.

1. Команда 
   ```
   openssl genrsa -out developer.key 2048
   ```
   генерирует новый приватный RSA-ключ длиной 2048 бит и сохраняет его в файл developer.key. Это ключ, используемый для обеспечения безопасности и подписями при создании сертификатов.

2. Команда 
   ```
   openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"
   ```
   создает новый запрос на подпись сертификата (CSR), используя приватный ключ developer.key. Параметр -subj задает субъект запроса, в данном случае Common Name (CN) равен "developer". CSR нужен для того, чтобы удостоверяющий центр (CA) мог проверить и подписать сертификат с указанными в нём данными.

3. Команда 
   ```
   openssl x509 -req -in developer.csr -CA /var/snap/microk8s/current/certs/ca.crt -CAkey /var/snap/microk8s/current/certs/ca.key -CAcreateserial -out developer.crt -days 365
   ```
   подписывает CSR файлом CA-сертификата и закрытым ключом CA, указанными в параметрах -CA и -CAkey. Параметр -CAcreateserial создает файл с серийным номером, если его нет. Результатом является подписанный сертификат developer.crt, действительный 365 дней. Этот процесс позволяет создать сертификат, который доверяет указанный CA.

<img width="1963" height="77" alt="Снимок экрана 2025-10-13 213830" src="https://github.com/user-attachments/assets/d25cb714-7908-4524-96b4-a73977e61961" />

### 3.3 Создаём Role (только просмотр логов и описания подов) и RoleBinding

https://github.com/DioRoman/kubernetes/blob/main/2.3/role-pod-reader.yaml

Данный объект Role в Kubernetes задает роль с именем "pod-viewer" в пространстве имен "default". 

Эта роль предоставляет права только на чтение (verbs: "get", "list", "watch") ресурсов "pods" (поды) и "pods/log" (логи подов) в указанном неймспейсе. То есть пользователи или сервисные аккаунты, которым привязана эта роль, могут просматривать информацию о подах и их логах, но не могут их изменять или удалять.

https://github.com/DioRoman/kubernetes/blob/main/2.3/rolebinding-developer.yaml

Данный объект RoleBinding с именем "developer-pod-viewer-binding" в пространстве имен "default" связывает пользователя с именем "developer" с ролью "pod-viewer".

<img width="768" height="104" alt="Снимок экрана 2025-10-13 224355" src="https://github.com/user-attachments/assets/5d5ca068-df06-47ae-846f-271cd52ad49c" />

### 3.4 Настраиваем kubectl для нового пользователя

`kubectl config set-cluster microk8s-cluster --server=https://10.152.183.1 --certificate-authority=/var/snap/microk8s/current/certs/ca.crt`

— Добавляет или обновляет в kubeconfig описание кластера с именем microk8s-cluster, указывая URL API-сервера Kubernetes (https://10.152.183.1) и путь к сертификату центра сертификации (CA), чтобы kubectl мог проверять SSL подключение к кластеру.

`kubectl config set-credentials developer --client-certificate=developer.crt --client-key=developer.key`

— Создаёт или обновляет в kubeconfig учётные данные пользователя с именем developer, указывая клиентский сертификат и ключ для аутентификации по сертификату.

`kubectl config set-context developer-context --cluster=microk8s-cluster --user=developer --namespace=default`

— Создаёт или обновляет в kubeconfig контекст developer-context, который связывает пользователя developer с кластером microk8s-cluster и указывает namespace по умолчанию default. Контекст хранит все необходимые данные для подключения kubectl.

`kubectl config use-context developer-context`

— Устанавливает активным контекстом developer-context. После этого все команды kubectl будут выполняться в контексте пользователя developer в указанном кластере и namespace.

<img width="1619" height="191" alt="Снимок экрана 2025-10-13 224525" src="https://github.com/user-attachments/assets/d7974f76-0626-465d-b6b3-b0cea4d7a375" />

### 3.5 Шаги выполнения

`microk8s enable ingress`

`microk8s kubectl apply -f deployment-ingress-tls.yaml`

`microk8s kubectl apply -f configmap-web.yaml`

`microk8s kubectl create secret tls myapp-tls --cert=tls.crt --key=tls.key`

`microk8s enable rbac`

`microk8s kubectl apply -f role-pod-reader.yaml`

`microk8s kubectl apply -f rolebinding-developer.yaml`

`kubectl config set-cluster microk8s-cluster --server=https://10.152.183.1 --certificate-authority=/var/snap/microk8s/current/certs/ca.crt`

`kubectl config set-credentials developer --client-certificate=developer.crt --client-key=developer.key`

`kubectl config set-context developer-context --cluster=microk8s-cluster --user=developer --namespace=default`

`kubectl config use-context developer-context`

### 3.6 Проверяем доступ пользователя

`kubectl config current-context`  # проверяем текущий используемый контекст и пользователя

`kubectl config view --minify`   # проверяем текущий используемый контекст и пользователя

`kubectl get pods`          # должно работать

`kubectl logs <pod-name>`    # должно работать

`kubectl create pod ...`     # должно быть запрещено

<img width="1045" height="968" alt="Снимок экрана 2025-10-13 220229" src="https://github.com/user-attachments/assets/1a94a05c-dc73-473c-a224-f7bde952a24a" />

