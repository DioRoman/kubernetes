cd /mnt/c/Users/rlyst/Netology/kubernetes/terraform
terraform init
terraform apply -auto-approve
terraform destroy -auto-approve

cd /mnt/c/Users/rlyst/Netology/kubernetes/ansible
ansible-playbook -i inventories/hosts.yml install-MicroK8S.yml
ansible-playbook -i inventories/hosts.yml install-kubectl.yml

microk8s enable dashboard

# Создать Pod
microk8s kubectl apply -f netology-web-pod.yaml

# Создать Service
microk8s kubectl apply -f netology-svc.yaml

# Проверить, что Pod работает
microk8s kubectl get pods netology-web --watch
# Остановить watch при статусе Running (Ctrl+C)

# Запустить порт-форвардинг для Service
microk8s kubectl port-forward service/netology-svc 8080:80
# Оставить терминал открытым

# В другом терминале выполнить curl для проверки
curl http://localhost:8080# Создать Pod
microk8s kubectl apply -f netology-web-pod.yaml

# Создать Service
microk8s kubectl apply -f netology-svc.yaml

# Проверить, что Pod работает
microk8s kubectl get pods netology-web --watch
# Остановить watch при статусе Running (Ctrl+C)

# Запустить порт-форвардинг для Service
microk8s kubectl port-forward service/netology-svc 8080:80
# Оставить терминал открытым

# В другом терминале выполнить curl для проверки
curl http://localhost:8080