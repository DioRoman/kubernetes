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

Запуск манифеста:

<img width="746" height="76" alt="Снимок экрана 2025-10-07 230543" src="https://github.com/user-attachments/assets/a9d4296c-14a6-4a4d-83ea-912ff6722d54" />

Демонстрация, того что контейнер multitool может читать данные из файла в смонтированной директории, в который busybox записывает данные каждые 5 секунд:

<img width="866" height="585" alt="Снимок экрана 2025-10-07 230617" src="https://github.com/user-attachments/assets/c17c9ebc-5218-4a71-8595-c319002ff32e" />

Удаление Deployment и PVC:

<img width="772" height="578" alt="Снимок экрана 2025-10-07 230948" src="https://github.com/user-attachments/assets/c1ad036d-57ff-40f3-97d8-d9e379e52097" />

Демонстрация, того что файл сохранился на локальном диске ноды после удаления PV:

<img width="506" height="307" alt="Снимок экрана 2025-10-07 231050" src="https://github.com/user-attachments/assets/3878cb91-eccb-4b83-9056-aacc580c419c" />

Файл остаётся после удаления PersistentVolume (PV), потому что в нашем PV используется тип тома hostPath, который просто монтирует локальную директорию с диска узла (ноды) Kubernetes в контейнер. Это не виртуальное или облачное хранилище, а реальная папка на файловой системе ноды. Политика persistentVolumeReclaimPolicy у PV установлена в Retain, что значит Kubernetes не удаляет физические данные автоматически при удалении PV или PVC. PV становится статусом Released, но данные остаются.

Список команд:

`kubectl apply -f pv-pvc.yaml`
`kubectl get pv`
`kubectl get pvc`
`kubectl get pods -l app=data-exchange-pvc`
`kubectl logs -f <pod_name> -c multitool`
`kubectl describe pod <pod_name>`
`kubectl delete deployment data-exchange-pvc`
`kubectl delete pvc local-pvc`
`kubectl describe pv local-pv`

## 3. StorageClass

Создаём Deployment приложения, использующего PVC, созданный на основе StorageClass.

https://github.com/DioRoman/kubernetes/blob/main/2.1/sc.yaml

Суть этого `Deployment` в том, чтобы развернуть один реплицированный под, внутри которого два контейнера обмениваются данными через общий том, подключённый как **PersistentVolume**.  

### Основная логика
- **Busybox (первый контейнер)** каждые 5 секунд дописывает строку с текущим временем в файл `exchange.txt` в директории `/data`.
- **Multitool (второй контейнер)** постоянно выполняет `tail -f` этого же файла, выводя новые строки в реальном времени.

### Хранение данных
- В поде смонтирован общий том **shared-data**, который связан с PVC (`local-pvc`), а тот — с PV (`local-pv`).
- PV использует `hostPath` `/mnt/data` на узле, что значит, что данные физически лежат на локальной директории ноды и сохраняются даже при перезапуске пода.
- StorageClass `local-storage` с `WaitForFirstConsumer` гарантирует, что привязка PV к PVC произойдёт только при создании первого потребителя (пода).

### Итог
Этот Deployment демонстрирует:
- работу **общего тома** между контейнерами в одном поде;
- использование **PersistentVolumeClaim** и **StorageClass** для управления хранилищем;
- пример обмена данными между контейнерами через файловую систему, причём данные сохраняются вне контейнеров.

Эти команды позволяют создать, проверить и удалить необходимые объекты для работы приложения с динамическим томом через StorageClass.

`kubectl apply -f sc.yaml`
`kubectl get storageclass`
`kubectl get pv`
`kubectl get pvc`
`kubectl get pods -l app=data-exchange-sc`
`kubectl logs -f <pod_name> -c multitool`
`kubectl describe pod <pod_name>`
`kubectl delete deployment data-exchange-sc`
`kubectl delete pvc local-pvc`
`kubectl describe pv local-pv`

