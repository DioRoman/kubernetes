## 1. Volume: обмен данными между контейнерами в поде

Создаём Deployment приложения, состоящего из контейнеров busybox и multitool.

https://github.com/DioRoman/kubernetes/blob/main/2.1/containers-data-exchange.yaml

Этот манифест Kubernetes описывает **Deployment**, создающий Pod с двумя контейнерами, которые обмениваются данными через общий том. Его основная суть — демонстрация обмена данными между контейнерами в одном Pod'е с помощью **emptyDir**.

### Разбор по компонентам

- **apiVersion: apps/v1** и **kind: Deployment**  
  Определяют, что создаётся контроллер типа *Deployment*, управляющий жизненным циклом подов (репликация, обновление, автозамена при сбое и т.д.).

- **metadata.name: data-exchange**  
  Имя деплоймента — `data-exchange`.

- **spec.replicas: 1**  
  Будет запущен один Pod.

- **spec.selector.matchLabels** и **template.metadata.labels**  
  Используются для связи Deployment с шаблоном Pod'а. Deployment будет управлять всеми подами с меткой `app: data-exchange`.

### Шаблон Pod’а (`template`)

- **volumes → emptyDir**  
  Создаётся временный том `shared-data`, который существует только во время жизни Pod'а. Он используется как общий каталог между контейнерами.

#### Контейнер 1: busybox
- Образ: `busybox`
- Команда: каждые 5 секунд записывает текущие дату и время в файл  
  `/data/exchange.txt`
- Монтирует том `shared-data` по пути `/data`

#### Контейнер 2: multitool (тоже busybox)
- Команда: отображает содержимое файла `/data/exchange.txt` в режиме реального времени с помощью `tail -f`
- Монтирует тот же том по тому же пути `/data`

### Результат работы

1. Контейнер `busybox` генерирует строку с текущим временем каждые 5 секунд и добавляет её в файл `exchange.txt`.  
2. Контейнер `multitool` мгновенно видит эти обновления и выводит их в лог.  
3. Они обмениваются данными через **общий том `emptyDir`**, который доступен обоим контейнерам, но не сохраняется после удаления Pod'а.

Таким образом, Deployment иллюстрирует концепцию *общих томов и обмена файлами между контейнерами внутри одного Pod’a* в Kubernetes.

<details><summary>kubectl describe pods data-exchange</summary>

```
ubuntu@kubernetes:~$ kubectl describe pods data-exchange
Name:             data-exchange-76d46fc476-65xcb
Namespace:        default
Priority:         0
Service Account:  default
Node:             kubernetes/10.0.2.15
Start Time:       Tue, 07 Oct 2025 17:22:05 +0000
Labels:           app=data-exchange
                  pod-template-hash=76d46fc476
Annotations:      cni.projectcalico.org/containerID: 581b95c826b7cd844b9d7ef9c74198c6c15d6326d6c2c1f00ee47f706b0febef
                  cni.projectcalico.org/podIP: 10.1.192.70/32
                  cni.projectcalico.org/podIPs: 10.1.192.70/32
Status:           Running
IP:               10.1.192.70
IPs:
  IP:           10.1.192.70
Controlled By:  ReplicaSet/data-exchange-76d46fc476
Containers:
  busybox:
    Container ID:  containerd://7d98d8bd4183d711cc2b1dbac379b2ce15f203a582ff927e0c92aa1f31b5c253
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:d82f458899c9696cb26a7c02d5568f81c8c8223f8661bb2a7988b269c8b9051e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while true; do
        echo "Current time: $(date)" >> /data/exchange.txt;
        sleep 5;
      done

    State:          Running
      Started:      Tue, 07 Oct 2025 17:22:10 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9z48q (ro)
  multitool:
    Container ID:  containerd://45e34e29704eb254911ed0ce3fd9540ee84af4732367e14d68080e1efe2704dd
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:d82f458899c9696cb26a7c02d5568f81c8c8223f8661bb2a7988b269c8b9051e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      tail -f /data/exchange.txt

    State:          Running
      Started:      Tue, 07 Oct 2025 17:22:11 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9z48q (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  shared-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  kube-api-access-9z48q:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  2m36s  default-scheduler  Successfully assigned default/data-exchange-76d46fc476-65xcb to kubernetes
  Normal  Pulling    2m36s  kubelet            Pulling image "busybox"
  Normal  Pulled     2m32s  kubelet            Successfully pulled image "busybox" in 3.427s (3.427s including waiting). Image size: 2223686 bytes.
  Normal  Created    2m32s  kubelet            Created container: busybox
  Normal  Started    2m32s  kubelet            Started container busybox
  Normal  Pulling    2m32s  kubelet            Pulling image "busybox"
  Normal  Pulled     2m31s  kubelet            Successfully pulled image "busybox" in 940ms (940ms including waiting). Image size: 2223686 bytes.
  Normal  Created    2m31s  kubelet            Created container: multitool
  Normal  Started    2m31s  kubelet            Started container multitool
```

</details>

Вывод команды `tail -f <имя общего файла>`

<img width="1155" height="275" alt="Снимок экрана 2025-10-07 202349" src="https://github.com/user-attachments/assets/32252a73-3812-4876-8383-7853a89505a6" />

## 2. PV, PVC

Создаём Deployment.

https://github.com/DioRoman/kubernetes/blob/main/2.1/pv-pvc.yaml

Суть данного манифеста — организация **совместного доступа двух контейнеров внутри одного пода к общему хранилищу**, которое сохраняет данные на локальном пути ноды через PersistentVolume (PV) и PersistentVolumeClaim (PVC).  

### Разбор по частям

#### PersistentVolume (PV)
- **apiVersion: v1, kind: PersistentVolume** — описание постоянного хранилища на стороне кластера.  
- `hostPath: /mnt/data` — используется локальная директория на хостовой машине (ноде Kubernetes).  
- `capacity: 1Gi` — вместимость PV составляет 1 ГБ.  
- `accessModes: ReadWriteOnce` — том может быть смонтирован для чтения и записи только одним подом.  
- `persistentVolumeReclaimPolicy: Retain` — при удалении PVC данные сохраняются на узле.  
- `storageClassName: local-storage` — указывает на класс хранения, который должен использоваться PVC.

#### PersistentVolumeClaim (PVC)
- **Запрашивает** использование PV размером 1 ГБ с тем же `storageClassName: local-storage`.  
- После связывания с PV предоставляет подам **абстрактный доступ** к реальному хранилищу.

#### Deployment
- **apiVersion: apps/v1, kind: Deployment** — описывает управление подами (репликами и их обновлением).  
- Создаёт один под (`replicas: 1`), состоящий из двух контейнеров:  
  - **busybox** — каждые 5 секунд записывает текущую дату и время в файл `/data/exchange.txt`.  
  - **multitool** (также busybox) — непрерывно читает и выводит содержимое этого же файла (`tail -f /data/exchange.txt`).  
- Оба контейнера **монтируют один и тот же том** `shared-data`, связанный с PVC `local-pvc`, в директорию `/data`.

### Итог
Манифест демонстрирует:
- создание локального постоянного хранилища (PV),
- использование этого хранилища подом через PVC,
- обмен данными между контейнерами в поде через общий том,
- сохранение данных на ноде даже после удаления пода (из-за `Retain`).

Скриншоты:

Запуск манифеста:

<img width="746" height="76" alt="Снимок экрана 2025-10-07 230543" src="https://github.com/user-attachments/assets/a9d4296c-14a6-4a4d-83ea-912ff6722d54" />

Демонстрация, того что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд:

<img width="866" height="585" alt="Снимок экрана 2025-10-07 230617" src="https://github.com/user-attachments/assets/c17c9ebc-5218-4a71-8595-c319002ff32e" />

Удаление Deployment и PVC:

<img width="772" height="578" alt="Снимок экрана 2025-10-07 230948" src="https://github.com/user-attachments/assets/c1ad036d-57ff-40f3-97d8-d9e379e52097" />

Демонстрация, того что файл сохранился на локальном диске ноды после удаления PV:

<img width="506" height="307" alt="Снимок экрана 2025-10-07 231050" src="https://github.com/user-attachments/assets/3878cb91-eccb-4b83-9056-aacc580c419c" />

Файл остаётся после удаления PersistentVolume (PV), потому что в нашем PV используется тип тома hostPath, который просто монтирует локальную директорию с диска узла (ноды) Kubernetes в контейнер. Это не виртуальное или облачное хранилище, а реальная папка на файловой системе ноды. Политика persistentVolumeReclaimPolicy у PV установлена в Retain, что значит Kubernetes не удаляет физические данные автоматически при удалении PV или PVC. PV становится статусом Released, но данные остаются.

## 3. StorageClass

Создаём Deployment приложения, использующего PVC, созданный на основе StorageClass.

https://github.com/DioRoman/kubernetes/blob/main/2.1/sc.yaml




<details><summary>kubectl describe pod kubectl data-exchange-pvc</summary>

```
  ubuntu@kubernetes:~$ kubectl describe pod kubectl data-exchange-pvc-7f79bb49c4-w2ph4
Name:             data-exchange-pvc-7f79bb49c4-w2ph4
Namespace:        default
Priority:         0
Service Account:  default
Node:             kubernetes/10.0.2.15
Start Time:       Tue, 07 Oct 2025 20:03:10 +0000
Labels:           app=data-exchange-pvc
                  pod-template-hash=7f79bb49c4
Annotations:      cni.projectcalico.org/containerID: 6f73ee5587b881b9b3f728a2484fca1879484bb413719b9b9f4afd39a2a9ab38
                  cni.projectcalico.org/podIP: 10.1.192.71/32
                  cni.projectcalico.org/podIPs: 10.1.192.71/32
Status:           Running
IP:               10.1.192.71
IPs:
  IP:           10.1.192.71
Controlled By:  ReplicaSet/data-exchange-pvc-7f79bb49c4
Containers:
  busybox:
    Container ID:  containerd://0daa387a47cfa275875289db76432eb02ed19e20bb2bba5a01fd1d8de24522ef
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:d82f458899c9696cb26a7c02d5568f81c8c8223f8661bb2a7988b269c8b9051e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      while true; do
        echo "Current time: $(date)" >> /data/exchange.txt;
        sleep 5;
      done

    State:          Running
      Started:      Tue, 07 Oct 2025 20:03:12 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gkw6b (ro)
  multitool:
    Container ID:  containerd://e43d972c62855d472b03b07e4909ea6b363b09ba4cae780419d170ae78058f4e
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:d82f458899c9696cb26a7c02d5568f81c8c8223f8661bb2a7988b269c8b9051e
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      tail -f /data/exchange.txt

    State:          Running
      Started:      Tue, 07 Oct 2025 20:03:13 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /data from shared-data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-gkw6b (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  shared-data:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  local-pvc
    ReadOnly:   false
  kube-api-access-gkw6b:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  5m6s  default-scheduler  0/1 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
  Normal   Scheduled         5m5s  default-scheduler  Successfully assigned default/data-exchange-pvc-7f79bb49c4-w2ph4 to kubernetes
  Normal   Pulling           5m4s  kubelet            Pulling image "busybox"
  Normal   Pulled            5m3s  kubelet            Successfully pulled image "busybox" in 978ms (978ms including waiting). Image size: 2223686 bytes.
  Normal   Created           5m3s  kubelet            Created container: busybox
  Normal   Started           5m3s  kubelet            Started container busybox
  Normal   Pulling           5m3s  kubelet            Pulling image "busybox"
  Normal   Pulled            5m2s  kubelet            Successfully pulled image "busybox" in 905ms (905ms including waiting). Image size: 2223686 bytes.
  Normal   Created           5m2s  kubelet            Created container: multitool
  Normal   Started           5m2s  kubelet            Started container multitool
Error from server (NotFound): pods "kubectl" not found
```

</details>

Мы создали PersistentVolume (PV), PersistentVolumeClaim (PVC) и Deployment с двумя контейнерами, чтобы обеспечить постоянное и совместное хранение данных в Kubernetes.

- **PV** — это ресурс хранения, выделенный в кластере, в нашем случае это папка на локальной ноде. Он существует независимо от жизненного цикла подов и служит абстракцией физического хранилища.

- **PVC** — это запрос пользователя или пода к PV с указанием требуемого объема и режима доступа. Kubernetes ищет подходящий PV, который соответствует требованиям PVC, и связывает их (binding).

- В нашем Deployment два контейнера — busybox записывает данные в файл в общем томе, а multitool читает этот файл.

Почему это так работает:

- Данные сохраняются в локальной директории ноды, связанной с PV, благодаря использованию `hostPath` в PV. Это обеспечивает устойчивость данных даже при рестарте или удалении подов.

- ReclaimPolicy установлен в `Retain`, поэтому при удалении PVC и Deployment PV не удаляется и данные остаются на диске.

- Связывание PVC и PV позволяет абстрагировать управление хранением от подов: они запрашивают нужный объем, а Kubernetes обеспечивает привязку и монтирование.

Таким образом, мы отделили управление данными от жизненного цикла контейнеров, гарантируя, что данные сохраняются и доступны для разных подов по мере необходимости.

Файл остаётся после удаления PersistentVolume (PV), потому что в нашем PV используется тип тома `hostPath`, который просто монтирует локальную директорию с диска узла (ноды) Kubernetes в контейнер. Это не виртуальное или облачное хранилище, а реальная папка на файловой системе ноды.

Основные причины, почему данные остаются:

- У PV с `hostPath` физические данные хранятся непосредственно в указанной директории на ноде (например, /mnt/data), и Kubernetes не управляет этим хранилищем напрямую — он только предоставляет доступ к нему контейнерам.

- Политика `persistentVolumeReclaimPolicy` у PV установлена в `Retain`, что значит Kubernetes не удаляет физические данные автоматически при удалении PV или PVC. PV становится статусом Released, но данные остаются.

- Удаление объекта PV освобождает ресурс в Kubernetes, но не удаляет файлы с локального диска, потому что Kubernetes не контролирует удаление папок или файлов на узле, это надо делать вручную.

Таким образом, с `hostPath` Kubernetes лишь "связывает" папку с подом, но не контролирует хранение или удаление данных, что и позволяет сохранять файлы даже после удаления PV. Чтобы удалить данные, нужно вручную очистить соответствующую директорию на ноде.


Вот последовательные команды для выполнения задания с StorageClass, PVC и Deployment:

1. Создать StorageClass из файла storageclass.yaml:
```bash
kubectl apply -f storageclass.yaml
```

2. Проверить, что StorageClass создан успешно:
```bash
kubectl get storageclass
```

3. Создать PersistentVolumeClaim из файла pvc.yaml (который ссылается на созданный StorageClass):
```bash
kubectl apply -f pvc.yaml
```

4. Проверить статус PVC:
```bash
kubectl get pvc
```

5. Создать Deployment из файла deployment.yaml, в котором контейнеры монтируют PVC:
```bash
kubectl apply -f deployment.yaml
```

6. Проверить статус подов:
```bash
kubectl get pods -l app=data-exchange-sc
```

7. Получить имя пода для чтения логов:
```bash
kubectl get pods -l app=data-exchange-sc -o jsonpath='{.items[0].metadata.name}'
```

8. Просмотреть логи контейнера multitool для проверки чтения файла:
```bash
kubectl logs -f <pod_name> -c multitool
```

9. Описать состояние PVC и PV (если нужно):
```bash
kubectl describe pvc local-pvc
kubectl describe pv
```

10. При необходимости удалить созданные ресурсы:
```bash
kubectl delete deployment data-exchange-sc
kubectl delete pvc local-pvc
kubectl delete storageclass local-storage
```

Эти команды выполнят полное создание и проверку работы StorageClass, PVC и Deployment для обмена файлами между двумя контейнерами через динамически выделенный локальный том.Вот последовательные команды для выполнения задания с StorageClass, PVC и Deployment:

1. Создать StorageClass:
```bash
kubectl apply -f storageclass.yaml
```

2. Проверить создание StorageClass:
```bash
kubectl get storageclass
```

3. Создать PVC:
```bash
kubectl apply -f pvc.yaml
```

4. Проверить статус PVC:
```bash
kubectl get pvc
```

5. Создать Deployment:
```bash
kubectl apply -f deployment.yaml
```

6. Проверить поды:
```bash
kubectl get pods -l app=data-exchange-sc
```

7. Получить имя пода:
```bash
kubectl get pods -l app=data-exchange-sc -o jsonpath='{.items[0].metadata.name}'
```

8. Просмотреть логи контейнера multitool:
```bash
kubectl logs -f <pod_name> -c multitool
```

9. При необходимости удалить ресурсы:
```bash
kubectl delete deployment data-exchange-sc
kubectl delete pvc local-pvc
kubectl delete storageclass local-storage
```

Эти команды обеспечат создание и проверку динамического тома с помощью StorageClass, PVC и обмен данными между контейнерами.Вот последовательные команды для создания и проверки StorageClass, PVC и Deployment в Kubernetes:

1. Создать StorageClass:
```bash
kubectl apply -f storageclass.yaml
```

2. Проверить StorageClass:
```bash
kubectl get storageclass
```

3. Создать PVC:
```bash
kubectl apply -f pvc.yaml
```

4. Проверить PVC:
```bash
kubectl get pvc
```

5. Создать Deployment:
```bash
kubectl apply -f deployment.yaml
```

6. Проверить под:
```bash
kubectl get pods -l app=data-exchange-sc
```

7. Получить имя пода:
```bash
kubectl get pods -l app=data-exchange-sc -o jsonpath='{.items[0].metadata.name}'
```

8. Просмотреть логи multitool:
```bash
kubectl logs -f <pod_name> -c multitool
```

9. Удалить ресурсы при необходимости:
```bash
kubectl delete deployment data-exchange-sc
kubectl delete pvc local-pvc
kubectl delete storageclass local-storage
```

Эти команды позволяют создать, проверить и удалить необходимые объекты для работы приложения с динамическим томом через StorageClass.
