# Пошаговая инструкция: установка кластера Kubernetes (1 мастер, containerd, etcd)

**Исходные данные:**
- Сервер: Ubuntu 24.04
- Внутренний IPv4: 10.0.2.22
- Публичный IPv4: 62.84.119.86
- CRI: containerd
- etcd – запускать на мастере

***

## 1. Подготовка сервера

**1.1. Настройте hostname и hosts:**
```bash
sudo hostnamectl set-hostname k8s-master
```

Добавьте в `/etc/hosts` (замените на ваши названия, если будут воркеры):
```
echo "10.0.2.26   k8s-master" | sudo tee -a /etc/hosts
```

**1.2. Отключите swap:**
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

**1.3. Загружайте необходимые модули ядра:**
```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

**1.4. Настройте параметры сети:**
```bash
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sudo sysctl --system
```

***

## 2. Установка containerd

**2.1. Установка компонентов:**
```bash
sudo apt update && sudo apt install -y containerd
```

**2.2. Инициализация default config:**
```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

**2.3. Включите поддержку systemd cgroup драйвера:**
В файле `/etc/containerd/config.toml` найдите секцию:
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
SystemdCgroup = true
```

**2.4. Перезапустите containerd:**
```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

***

## 3. Установка Kubernetes (kubeadm, kubelet, kubectl)

**3.1. Добавьте репозиторий:**
```bash
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
```

**3.2. Установите компоненты:**
```bash
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

***

## 4. Инициализация кластера (etcd на мастере)

**4.1. Инициализация кластера:**
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///run/containerd/containerd.sock
```
- Флаг `--cri-socket` указывает containerd как CRI.
- etcd на мастере запускается по умолчанию.

**4.2. Настройте доступ к кластеру:**
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

***

## 5. Установка сетевого плагина CNI (например, Flannel)

**5.1. Установите Flannel:**
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

***

## 6. Проверка

**6.1. Проверьте состояние узлов и компонентов:**
```bash
kubectl get nodes
kubectl get pods -A
```

***

### Краткий итог
- На сервере Ubuntu 24.04 установлен Kubernetes (1 мастер-нода), runtime — containerd, etcd встроенный на мастере по умолчанию.
- После выполнения этих шагов кластер готов к добавлению воркеров и развертыванию приложений.

***
