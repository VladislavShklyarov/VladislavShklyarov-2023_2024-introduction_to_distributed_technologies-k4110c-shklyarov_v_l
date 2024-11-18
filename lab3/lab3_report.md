Faculty: [FICT](https://fict.itmo.ru)<br>
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)<br>
Year: 2024/2025<br>
Group: K4110c<br>
Author: Shklyarov Vladislav Leonidovich<br>
Lab: Lab3<br>
Date of create: 06.11.2024<br>
Date of finished: 14.11.2024<br>

###Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube.

###Ход работы
1) Создать `configMap` с переменными `REACT_APP_USERNAME`, `REACT_APPCOMPANY_NAME`.
2) Создать `replicaSet` с 2 репликами контейнера и передать переменные `REACT_APP_USERNAME`, `REACT_APPCOMPANY_NAME` через `configMap`.
3) Включить `minikube addons enable ingress` и сгенерировать TLS сертификат, импортировать сертификат в minikube.
4) Создать `ingress` в minikube
5) Прописать FQDN и IP адресс ingress в `hosts` и перейти в браузер.
6) Войти в веб-приложение по собственному FQDN используя HTTPS и проверить наличие сертификата.

### Выполнение работы
#### 1. Создание `configMap` с переменными `REACT_APP_USERNAME`, `REACT_APPCOMPANY_NAME`.
`configMap` это объект Kubernetes, который используется для хранение данных о конфигурации. С его помощью можно отделять конфигурацию приложения от самого кода. Это позволяет использовать один и тот же код для разных окружений (тестирования, разработки, прод) не изменяя его и хранить переменные среды.
Для начала создадим новый кластер minikube с названием configmap-cluster.
```bash
minikube start --profile=configmap-cluster
minikube profile list
kubectl config use-context configmap-cluster

```
