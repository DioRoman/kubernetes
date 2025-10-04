Этот набор команд используется для работы с Terraform, Ansible и MicroK8s (локальной Kubernetes-средой).
Вот краткое описание команд:

`cd /mnt/c/Users/rlyst/Netology/kubernetes/terraform`

### Terraform
- `terraform init` — инициализация рабочего каталога Terraform (загрузка провайдеров, настройка).
- `terraform apply -auto-approve` — применение конфигураций Terraform без запроса подтверждения (создание инфраструктуры).
- `terraform destroy -auto-approve` — удаление всей инфраструктуры, созданной через Terraform, без подтверждения.

`cd /mnt/c/Users/rlyst/Netology/kubernetes/ansible`

### Ansible
- `ansible-playbook -i /mnt/c/Users/rlyst/Netology/kubernetes/ansible/inventories/hosts.yml /mnt/c/Users/rlyst/Netology/kubernetes/ansible/install-MicroK8S.yml` — запуск Ansible-плейбука для установки MicroK8s на хостах из файла инвентаризации.
- `ansible-playbook -i inventories/hosts.yml install-kubectl.yml` — запуск плейбука для установки kubectl на целевых машинах.

### MicroK8s и Kubernetes
- `microk8s enable dashboard` — включение Kubernetes Dashboard.
- `microk8s kubectl apply -f netology-web-pod.yaml` — создание Pod из YAML-манифеста.
- `microk8s kubectl apply -f netology-svc.yaml` — создание Service для Pod.
- `microk8s kubectl get pods netology-web --watch` — наблюдение за состоянием Pod до статуса Running (Ctrl+C для остановки).
- `microk8s kubectl port-forward service/netology-svc 8080:80` — проброс локального порта 8080 к порту 80 сервиса Kubernetes для локального доступа.
- `curl http://localhost:8080` — проверка доступности приложения через локальный порт.

### Управление Pod и Deployment
- `microk8s.kubectl delete pod multitool-nginx-7cc954d566-v4skf` — удаление конкретного Pod.
- `microk8s.kubectl scale deployment/nginx-multitool --replicas=2` — изменение количества реплик у deployment.
- `microk8s.kubectl apply -f deployment.yaml` — создание или обновление Deployment по YAML-манифесту.
- `microk8s.kubectl apply -f service-nodeport.yaml` — создание или обновление service по YAML-манифесту.
- `microk8s.kubectl apply -f service-clusterip.yaml` — создание или обновление service по YAML-манифесту.
- `microk8s kubectl exec -it multitool-test -- /bin/sh` — запуск интерактивной командной оболочки внутри контейнера Pod multitool-test.
- `microk8s.kubectl get pods` - проверка состояния pods

kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh

curl nginx-multitool-service:80
curl nginx-multitool-service:8080

curl frontend-service:80