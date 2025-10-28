Вот единообразный и структурированный список команд с кратким описанием для Terraform, Ansible и MicroK8s (Kubernetes):

***

### Terraform

- `cd /mnt/c/Users/rlyst/Netology/kubernetes/terraform` — переход в каталог с конфигурациями Terraform.
- `terraform init` — инициализация рабочего каталога Terraform, загрузка необходимых провайдеров.
- `terraform apply -auto-approve` — применение конфигураций Terraform без запроса подтверждения, создание инфраструктуры.
- `terraform destroy -auto-approve` — удаление созданной инфраструктуры автоматически, без подтверждения.

***

### Ansible

- `cd /mnt/c/Users/rlyst/Netology/kubernetes/ansible` — переход в каталог с Ansible-плейбуками.
- `ansible-playbook -i inventories/hosts.yml install-MicroK8S.yml` — запуск плейбука для установки MicroK8s на указанных хостах.
- `ansible-playbook -i inventories/hosts.yml install-master.yml` — запуск плейбука для установки MicroK8s на указанных хостах.
- `ansible-playbook -i inventories/hosts.yml install-node.yml` — запуск плейбука для установки MicroK8s на указанных хостах.

***

### MicroK8s и kubectl

- `microk8s enable dashboard` — включение Kubernetes Dashboard для визуального управления кластером.
- `microk8s kubectl apply -f <файл.yaml>` — применение конфигураций из YAML-манифеста (создание/обновление объектов Kubernetes).
  - Примеры файлов: `netology-web-pod.yaml`, `netology-svc.yaml`, `deployment.yaml`, `service-nodeport.yaml`, `service-clusterip.yaml`, `deployment-frontend.yaml`, `deployment-backend.yaml`, `service-frontend.yaml`, `service-backend.yaml`, `ingress.yaml`.
- `microk8s kubectl get pods` — просмотр списка Pod'ов и их состояния.
- `microk8s kubectl get pods <имя-pod> --watch` — мониторинг состояния конкретного Pod до статуса Running (Ctrl+C для остановки).
- `microk8s kubectl delete pod <pod-name>` — удаление указанного Pod.
- `microk8s kubectl scale deployment/<deployment-name> --replicas=<число>` — изменение количества реплик Deployment.
- `microk8s kubectl exec -it <pod-name> -- /bin/sh` — запуск интерактивной оболочки внутри контейнера Pod.
- `microk8s kubectl port-forward service/<service-name> <локальный-порт>:<порт-сервиса>` — проброс локального порта к сервису Kubernetes.

***

### Тестирование и отладка

- `kubectl run test-pod --image=wbitt/network-multitool --rm -it -- sh` — запуск временного Pod с сетьевым инструментарием для тестирования.
- Внутри тестового Pod или на локальной машине использовать команды:
  - `curl <service-name>:<порт>` — проверка доступности сервисов внутри кластера.
  - Примеры: `curl nginx-multitool-service:80`, `curl nginx-multitool-service:8080`, `curl frontend-service:80`.
- `curl http://localhost:8080` — проверка локального доступа к проброшенному порту.
- `curl http://62.84.116.85/` и `curl http://62.84.116.85/api` — запросы к наружному IP для проверки работы Ingress.

***

### Ingress (входящий трафик)

- `microk8s enable ingress` — включение компонента Ingress.
- `microk8s kubectl get ingress` — просмотр настроек и статуса Ingress.
- `microk8s kubectl get pods -n ingress` — просмотр Pod'ов, связанных с Ingress контроллером.


sudo containerd config default > /etc/containerd/config.toml
sudo rm -rf /var/run/containerd/containerd.sock

sudo kubeadm reset
sudo kubeadm init --apiserver-advertise-address=10.0.2.18 --pod-network-cidr=10.244.0.0/16 --apiserver-cert-extra-sans=51.250.92.109,178.154.234.213 --control-plane-endpoint=10.0.2.18
export KUBECONFIG=/etc/kubernetes/admin.conf
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p

kubectl get nodes

kubectl get pods -A
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

ansible-playbook -i inventories/hosts.yml install-node.yml --extra-vars "kube_join_command='kubeadm join k8s-master:6443 --token 9097wk.89sxdg42ppeaka0b --discovery-token-ca-cert-hash sha256:c570a89de9a8c154757d606651a230edf9444549f0c3d482099adf5506392d3a'"