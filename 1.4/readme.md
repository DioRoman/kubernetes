## Описание прооекта:

Проект создает среду с двумя приложениями — backend и frontend — в Kubernetes через MicroK8s, с автоматической маршрутизацией и балансировкой запросов с помощью Ingress, чтобы обеспечить доступность и масштабируемость.

***

## Описание компонентов:

- **deployment-backend.yaml** — создаёт деплоймент с одним подом, который запускает контейнер с образом `wbitt/network-multitool`. В контейнере открыт порт 8080, переменная окружения задаёт порт `HTTP_PORT`=8080.
- **deployment-frontend.yaml** — создаёт деплоймент для frontend с nginx, который слушает порт 80.
- **service-backend.yaml** — создаёт сервис для backend, который проксирует на порт 8080 в подах с меткой `app: backend`.
- **service-frontend.yaml** — сервис для фронтенда, проксирующий на порт 80 подов с меткой `app: frontend`.
- **ingress.yaml** — объект Ingress для маршрутизации HTTP-запросов. Запросы к `/` идут к `frontend-service` на порт 80, а к `/api` — к `backend-service` на порт 8080.

***

## Команды для запуска через MicroK8s:

1. Включите аддон Ingress (если ещё не включён):
```
microk8s enable ingress
```

2. Примените все манифесты:
```
microk8s kubectl apply -f deployment-backend.yaml
microk8s kubectl apply -f deployment-frontend.yaml
microk8s kubectl apply -f service-backend.yaml
microk8s kubectl apply -f service-frontend.yaml
microk8s kubectl apply -f ingress.yaml
```


## Проверка работы:

1. Посмотрите состояние подов:
```
microk8s kubectl get pods
```
Убедитесь, что все в состоянии `Running`.

2. Посмотрите сервисы:
```
microk8s kubectl get svc
```

3. Посмотрите ingress:
```
microk8s kubectl get ingress
```

4. Проверьте доступ к фронтенду, в браузере или через curl:
```
curl http://<IP_вашего_кластера>/
```
Должен отобразиться nginx.

5. Проверьте доступ к backend API:
```
curl http://<IP_вашего_кластера>/api
```
Вы получите ответ от network-multitool.

Если IP-адрес кластера — это localhost (например, MicroK8s на вашей машине), обращайтесь на http://localhost/ и http://localhost/api соответственно.

***