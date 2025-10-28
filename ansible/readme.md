1. Разворачиваем 1 мастер и 4 нода в YC через Terraform

2. Запускаем установку, настройку и инициализацию Master через ansible-playbook

3. На мастер вводим команду

kubeadm token create --print-join-command

полученный код поставляем как extra vars при запуске 

4. Запускаем установку, настройку и инициализацию Node через ansible-playbook