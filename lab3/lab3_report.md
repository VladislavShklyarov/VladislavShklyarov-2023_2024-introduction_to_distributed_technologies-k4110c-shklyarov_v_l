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
Для начала создадим новый кластер minikube с названием configmap-cluster и переключимя на него.
```bash
minikube start --profile=configmap-cluster
minikube profile list
kubectl config use-context configmap-cluster
```
![image](https://github.com/user-attachments/assets/729b7e75-cfdc-4834-87eb-6678b4db94fc)

Cудя по выводу в консоли у нас все получилось. Теперь можно переходить к созданию `configMap`.
Для этого создадим соответсвующий манифест.

![image](https://github.com/user-attachments/assets/e5b8ce00-92d4-4a9b-bc12-a2c646fc72b9)

1. В параметре `kind` мы указываем соответсвующий тип объекта: `ConfigMap`.
2. В разделе `data` создаем переменные `REACT_APP_USERNAME`, `REACT_APPCOMPANY_NAME` и определяем их.

Теперь можно применить манифест и проверить, что все прошло успешно:

```bash
kubectl apply -f configmap.yaml
kubectl get configmap react-config -o yaml

```

![image](https://github.com/user-attachments/assets/edc14d0f-3b86-4ade-b99a-c326d8571801)

Все прошло без ошибок, `konfigMap` создан.

####2. Создание `replicaSet` с 2 репликами контейнера.
####Передача переменных `REACT_APP_USERNAME`, `REACT_APPCOMPANY_NAME` через `configMap`.

В отличии от похожего объекта `deployment` из предыдущей лабораторной работы, `replicaSet` представляет собой более "низкий" уровень управления контейнерами. Он не поддерживает обновления, имеет более простую структуру, а его основной задачей является поддержка стабильного числа подов. `deployment` имеет более высокий уровень абстракции, он позволяет осуществлять управления обновлениями и оптимизацию жизненного цикла приложений. Кроме того, `replicaSet` создаются автоматически при создании `deployment`, и являются неотъемлимой его частью.

Для его создания также нужно написать манифест.

