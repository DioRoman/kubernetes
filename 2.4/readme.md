## Как использовать

1. Развернуть helm чарт с настройками по умолчанию:

```bash
helm install my-wordpress ./wordpress-chart
```

2. Чтобы изменить версию приложения, например Wordpress или MariaDB, изменить теги образов в `values.yaml` либо указать при установке:

```bash
helm install my-wordpress ./wordpress-chart --set wordpress.image.tag=5.9 --set mariadb.image.tag=10.6
```

***

Данный шаблон обеспечивает разделение компонентов Wordpress и MariaDB в отдельные деплойменты/стейтфулсеты, а также параметризацию образов через values.yaml для удобного управления версиями и конфигурациями под разные окружения. Если нужны дополнительные компоненты (например, ingress, pvc для Wordpress), их можно добавлять аналогично.