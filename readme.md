# MicroK8s Pod and Service Demo

## Описание проекта
Демонстрация базовых операций с MicroK8s — легковесным Kubernetes для локальной разработки и тестирования. Проект включает создание Pod с эхо-сервером, настройку Service для подключения к Pod и организацию локального доступа через порт-форвардинг.

---

## Дополнительно

### Terraform
Использован Terraform код для автоматического развертывания инфраструктуры в Yandex.Cloud:

https://github.com/DioRoman/kubernetes/blob/main/terraform/main.tf

- Создание виртуальной частной сети (VPC) с подсетями и группами безопасности.
- Разворачивание виртуальных машин (VM) с параметрами CPU, памяти и диска.
- Настройка VM для Kubernetes с помощью пользовательских данных.

<img width="1732" height="290" alt="Снимок экрана 2025-09-16 203226" src="https://github.com/user-attachments/assets/47c92e65-bbd2-4612-98bb-6d9c74ebfb20" />

### Ansible Playbook
Представлен Ansible playbook для установки и настройки MicroK8s на Ubuntu 24.04:

https://github.com/DioRoman/kubernetes/blob/main/ansible/install-MicroK8S.yml

- Установка snapd и MicroK8s из канала stable.
- Добавление пользователя к группе microk8s.
- Запуск и подготовка MicroK8s, включение dashboard.
- Настройка публичного порта для дашборда с автофорвардингом.
- Копирование kubeconfig для работы с kubectl от имени пользователя.

<img width="2232" height="1153" alt="Снимок экрана 2025-09-16 225718" src="https://github.com/user-attachments/assets/914ffde9-1d07-4516-82d9-e36a1fef835d" />

---

## Требования

| Требование                 | Описание                            |
|----------------------------|------------------------------------|
| ОС                         | Linux (рекомендуется Ubuntu 24.04) |
| MicroK8s                   | Установлен и запущен                |
| Пользователь               | В группе `microk8s`                 |
| Kubectl                    | Установлен или используется встроенный в MicroK8s |

---
