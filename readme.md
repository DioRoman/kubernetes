cd /mnt/c/Users/rlyst/Netology/kubernetes/terraform
terraform init
terraform apply -auto-approve
terraform destroy -auto-approve

cd /mnt/c/Users/rlyst/Netology/kubernetes/ansible
ansible-playbook -i inventories/hosts.yml install-MicroK8S.yml
ansible-playbook -i inventories/hosts.yml install-kubectl.yml

microk8s enable dashboard