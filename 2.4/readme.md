
## Перед работой

 Добавляем в ansible-playbook:

    - установку Helm
    - Включение аддона Ingress
    - копирование Helm-чарт в папку пользователя.

https://github.com/DioRoman/kubernetes/blob/main/ansible/install-MicroK8S.yml

## 1. Подготовливаем Helm-чарт для приложения

### 1.1 Упаковываем приложение myapp в Helm-чарт для деплоя в разные окружения.

myapp/  
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

https://github.com/DioRoman/kubernetes/tree/main/myapp

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

<img width="602" height="159" alt="Снимок экрана 2025-10-19 145353" src="https://github.com/user-attachments/assets/01d3e0aa-d298-4daa-a4d9-c28f8cfaba58" />

Проверяем работу:

`kubectl get pods`

`kubectl get svc`

`kubectl get ingress`

<img width="862" height="216" alt="Снимок экрана 2025-10-19 145402" src="https://github.com/user-attachments/assets/530bba01-7bd1-4b62-bc87-5ef27f9ad89f" />

<img width="595" height="84" alt="Снимок экрана 2025-10-19 145405" src="https://github.com/user-attachments/assets/3b595853-0b50-4254-a767-8100c49e79c8" />

В переменных Helm-чарт изменияем образ приложения для изменения версии.

`help upgrade my-app .\myapp`

<img width="545" height="193" alt="Снимок экрана 2025-10-19 145840" src="https://github.com/user-attachments/assets/f7a57cca-8ded-4236-a46b-fd1abee9c0d6" />

Проверяем работу

`curl http://ip`

<img width="1147" height="61" alt="Снимок экрана 2025-10-19 150213" src="https://github.com/user-attachments/assets/72a1d9ae-36f4-4cad-9755-a4a8613caabb" />

## 2. Запускаем две версии в разных неймспейсах

### 2.1 Создаём namespaces

`kubectl create namespace app1`

`kubectl create namespace app2`

<img width="517" height="97" alt="Снимок экрана 2025-10-19 172620" src="https://github.com/user-attachments/assets/b2bf7daa-b888-43af-86a5-c4bfb2aa2406" />

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
<img width="621" height="695" alt="Снимок экрана 2025-10-19 173641" src="https://github.com/user-attachments/assets/d2450223-c63f-4a57-9fe6-42ba33a428c7" />

### 2.3 Результат

Посмотреть список всех релизов можно командой

`helm list -A`

<img width="1446" height="126" alt="Снимок экрана 2025-10-19 151014" src="https://github.com/user-attachments/assets/fdb04c64-cb9e-4263-b979-80fc6b098159" />

Просмотреть поды

`kubectl get pods -n app1`

`kubectl get pods -n app2`

<img width="794" height="235" alt="Снимок экрана 2025-10-19 172805" src="https://github.com/user-attachments/assets/62cd55bc-71c9-4179-ad47-2e96b545624d" />
