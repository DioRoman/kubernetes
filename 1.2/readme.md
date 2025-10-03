# MicroK8s Pod and Service Demo

## Описание проекта
Демонстрация базовых операций с MicroK8s — легковесным Kubernetes для локальной разработки и тестирования. Проект включает создание Pod с эхо-сервером, настройку Service для подключения к Pod и организацию локального доступа через порт-форвардинг.

---

## Требования

| Требование                 | Описание                            |
|----------------------------|------------------------------------|
| ОС                         | Linux (рекомендуется Ubuntu 24.04) |
| MicroK8s                   | Установлен и запущен                |
| Пользователь               | В группе `microk8s`                 |
| Kubectl                    | Установлен или используется встроенный в MicroK8s |

---

## Создание Pod `hello-world`

1. Создайте файл `hello-world-pod.yaml`:

```
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  - name: echoserver
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 8080
```

2. Примените манифест:

```
microk8s kubectl apply -f hello-world-pod.yaml
```

3. Проверьте статус Pod:

```
microk8s kubectl get pods hello-world --watch
```

4. Запустите порт-форвардинг:

```
microk8s kubectl port-forward pod/hello-world 8080:8080
```

5. Проверьте доступ:

```
curl http://localhost:8080
```

---

<img width="801" height="220" alt="Снимок экрана 2025-09-16 202324" src="https://github.com/user-attachments/assets/46febe8d-77f9-4312-8368-8cad324b524e" />

<img width="552" height="638" alt="Снимок экрана 2025-09-16 202250" src="https://github.com/user-attachments/assets/1bedb152-9b8a-41f4-af55-785245472c1d" />

<img width="1795" height="879" alt="Снимок экрана 2025-09-16 202158" src="https://github.com/user-attachments/assets/54f81231-e1f7-4dc5-8f42-ac86430af996" />

<img width="1790" height="944" alt="Снимок экрана 2025-09-16 202225" src="https://github.com/user-attachments/assets/5114fb27-33a7-4b32-a11c-db65c1b51e3b" />

## Создание Pod `netology-web` и Service `netology-svc`

1. Создайте файл `netology-web-pod.yaml`:

```
apiVersion: v1
kind: Pod
metadata:
  name: netology-web
  labels:
    app: netology-web
spec:
  containers:
  - name: echoserver
    image: gcr.io/kubernetes-e2e-test-images/echoserver:2.2
    ports:
    - containerPort: 8080
```

2. Создайте файл `netology-svc.yaml`:

```
apiVersion: v1
kind: Service
metadata:
  name: netology-svc
spec:
  selector:
    app: netology-web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

3. Примените конфигурации:

```
microk8s kubectl apply -f netology-web-pod.yaml
microk8s kubectl apply -f netology-svc.yaml
```

4. Проверьте статус Pod:

```
microk8s kubectl get pods netology-web --watch
```

5. Запустите порт-форвардинг службы:

```
microk8s kubectl port-forward service/netology-svc 8080:80
```

6. Проверьте доступ:

```
curl http://localhost:8080
```

<img width="531" height="628" alt="Снимок экрана 2025-09-16 202921" src="https://github.com/user-attachments/assets/3e725b91-258b-4998-a274-f4c51449aa54" />

<img width="1785" height="973" alt="Снимок экрана 2025-09-16 202937" src="https://github.com/user-attachments/assets/359c7ac5-1d12-42c2-9f6f-3a1fe1b2b00d" />

<img width="1792" height="559" alt="Снимок экрана 2025-09-16 202948" src="https://github.com/user-attachments/assets/a105f023-5da8-4cbb-8990-56f397d6e3a8" />

---

## Итоги

- Автоматизация инфраструктуры с помощью Terraform для Kubernetes-узлов
- Автоматическая установка и настройка MicroK8s через Ansible
- Создание и запуск Pod и Service в MicroK8s
- Использование порт-форвардинга для локального тестирования
