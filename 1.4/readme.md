## Настройка Service (ClusterIP и NodePort)

## Создан Deployment с двумя контейнерами.

https://github.com/DioRoman/kubernetes/blob/main/1.3/deployment.yaml

<img width="2492" height="964" alt="Снимок экрана 2025-10-04 211602" src="https://github.com/user-attachments/assets/0b558400-f944-4a80-b404-2ece22da027c" />

## Создан Service типа ClusterIP

https://github.com/DioRoman/kubernetes/blob/main/1.4/service-clusterip.yaml

<img width="2493" height="507" alt="Снимок экрана 2025-10-04 211612" src="https://github.com/user-attachments/assets/a3f90dd7-33e5-47a3-aaf4-65d766a4eb4d" />

## Проверена доступность изнутри кластера

<img width="1619" height="698" alt="Снимок экрана 2025-10-04 205411" src="https://github.com/user-attachments/assets/54dbc02c-d03f-427c-a1ef-734b51a9cf48" />

##  Создан Service типа NodePort 

https://github.com/DioRoman/kubernetes/blob/main/1.4/service-nodeport.yaml

<img width="2489" height="745" alt="Снимок экрана 2025-10-04 212946" src="https://github.com/user-attachments/assets/83d0b39a-4250-4f3e-a581-e619b943ef8b" />

##  Проверен доступ с локального компьютера

<img width="1510" height="722" alt="Снимок экрана 2025-10-04 215131" src="https://github.com/user-attachments/assets/ef1d27c0-4408-4dec-9072-06a53488569f" />

## Настройка Ingress:

Проект создает среду с двумя приложениями — backend и frontend — в Kubernetes через MicroK8s, с автоматической маршрутизацией и балансировкой запросов с помощью Ingress, чтобы обеспечить доступность и масштабируемость.

***

## Описание компонентов:

- **deployment-backend.yaml** — создаёт деплоймент с одним подом, который запускает контейнер с образом `wbitt/network-multitool`. В контейнере открыт порт 8080, переменная окружения задаёт порт `HTTP_PORT`=8080.

https://github.com/DioRoman/kubernetes/blob/main/1.4/deployment-backend.yaml

- **deployment-frontend.yaml** — создаёт деплоймент для frontend с nginx, который слушает порт 80.

https://github.com/DioRoman/kubernetes/blob/main/1.4/deployment-frontend.yaml

- **service-backend.yaml** — создаёт сервис для backend, который проксирует на порт 8080 в подах с меткой `app: backend`.

https://github.com/DioRoman/kubernetes/blob/main/1.4/service-backend.yaml

- **service-frontend.yaml** — сервис для фронтенда, проксирующий на порт 80 подов с меткой `app: frontend`.

https://github.com/DioRoman/kubernetes/blob/main/1.4/service-frontend.yaml

- **ingress.yaml** — объект Ingress для маршрутизации HTTP-запросов. Запросы к `/` идут к `frontend-service` на порт 80, а к `/api` — к `backend-service` на порт 8080.

https://github.com/DioRoman/kubernetes/blob/main/1.4/ingress.yaml

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

<img width="822" height="557" alt="Снимок экрана 2025-10-05 204156" src="https://github.com/user-attachments/assets/3ac73ab3-7149-44f6-975f-64dfbf7b0218" />

5. Проверьте доступ к backend API:
```
curl http://<IP_вашего_кластера>/api
```
Вы получите ответ от network-multitool.

<img width="1423" height="54" alt="Снимок экрана 2025-10-05 204202" src="https://github.com/user-attachments/assets/0c962db2-b7a3-476f-a2f5-08a4760a4a24" />

Если IP-адрес кластера — это localhost (например, MicroK8s на вашей машине), обращайтесь на http://localhost/ и http://localhost/api соответственно.

***
