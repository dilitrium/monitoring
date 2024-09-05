# monitoring

Спринт 3

ЗАДАЧА

```
1. Настройка сборки логов.

Представьте, что вы разработчик, и вам нужно оперативно получать информацию с ошибками работы приложения.
Выберите инструмент, с помощью которого такой функционал можно предоставить. 
Нужно собирать логи работы пода приложения. Хранить это всё можно либо в самом кластере Kubernetes, либо на srv-сервере.

2. Выбор метрик для мониторинга.

Так, теперь творческий этап. Допустим, наше приложение имеет для нас некоторую важность. 
Мы бы хотели знать, когда пользователь не может на него попасть — время отклика, сертификат, статус код и так далее. 
Выберите метрики и инструмент, с помощью которого будем отслеживать его состояние.
Также мы хотели бы знать, когда место на srv-сервере подходит к концу.
Важно! Весь мониторинг должен находиться на srv-сервере, чтобы в случае падения кластера мы все равно могли узнать об этом.

3. Настройка дашборда.

Ко всему прочему хотелось бы и наблюдать за метриками в разрезе времени. 
Для этого мы можем использовать Grafana и Zabbix — что больше понравилось.

4. Алертинг.

А теперь добавим уведомления в наш любимый мессенджер, точнее в ваш любимый мессенджер. 
Обязательно протестируйте отправку уведомлений. Попробуйте «убить» приложение самостоятельно, и засеките время от инцидента до получения уведомления. 
Если время адекватное, то можно считать, что вы справились с этим проектом!
```

РЕШЕНИЕ

  - Для сборки диагностической информации будем использовать Loki: https://artifacthub.io/packages/helm/grafana/loki?modal=install
  - Добавляем репозиторий Loki и перечитываем репозитории:
  ```
  helm repo add bitnami https://charts.bitnami.com/bitnami -n monitoring && helm repo update
  ```
  Устанавливаем Loki в кластер k8s из helm chart:
  ```
  helm install --namespace monitoring loki bitnami/grafana-loki --set global.dnsService=coredns --set spec.type=NodePort --set loki.auth_enabled=false
  ```

![install loki](https://github.com/dilitrium/screendiplom/blob/58fa7195c6862fb34467a9fcd63ced15fde2e030/monitoring/loki_repos.png)
![install loki](https://github.com/dilitrium/screendiplom/blob/58fa7195c6862fb34467a9fcd63ced15fde2e030/monitoring/loki_install.png)


  - Для мониторинга кластера и приложения будем использовать Prometheus stack: https://artifacthub.io/packages/helm/prometheus-community/prometheus?modal=install

  Стэк мониторинга - Grafana\Prometheus\Blackbox\Node Exporter\Alertmanager\Loki
  - В каталоге prometheus_stack описываем через docker compose весь наш стэк мониторинга
  - Находясь в каталоге prometheus_stack с файлом docker-compose.yml запускаем развёртывание стэка:
  ```
  docker compose up -d 
  ```
  ``
![docker ps](https://github.com/dilitrium/screendiplom/blob/adc2f8a3b6dcb7eacda7192b1fe6897b7d53579c/monitoring/docker_grafana.png)

 - Подключаем как сервер srv, так и кластер k8s к визуализации метрик и логирования в единой Grafana:

![datasource](https://github.com/vajierik/monitoring/assets/150177457/09534642-4173-4d75-892c-7b2e9eccc730)

- Из registry готовых дэшбордов: https://grafana.com/grafana/dashboards/ выбираем нужные нам дэшборды и по id ставим.

![node_exporter](https://github.com/vajierik/monitoring/assets/150177457/fe7bd92d-85d6-4037-ad62-76b8c4b5ae60)

![prometheus](https://github.com/vajierik/monitoring/assets/150177457/4ab61133-013f-44a1-9626-b8e3a12ca20b)

- Логи нашего приложения из этой же самой grafana:

![logs-app](https://github.com/vajierik/monitoring/assets/150177457/e00e8c7e-4b42-4f54-bcd1-503684232813)

- Адрес grafana для просмотра логов и визуализированных метрик:

  http://51.250.86.66:3000/
  
  - В docker-compouse.yaml файле на сервере srv вносим свои данные телеграмм, перезапускаем контейнер и моделируем срабатывание аллерта

![instansedown](https://github.com/vajierik/monitoring/assets/150177457/8c79df55-e18b-4033-a850-ae1fe51a407e)

![replicas 0](https://github.com/vajierik/monitoring/assets/150177457/ac861e66-717f-4c33-82dd-4c4824926d11)

![instanseup](https://github.com/vajierik/monitoring/assets/150177457/f203b0a5-e14c-428f-8ab1-68f05e9a95bb)

![resolved](https://github.com/vajierik/monitoring/assets/150177457/42dbd018-8a59-4a07-9999-c83981ca2a7e)
