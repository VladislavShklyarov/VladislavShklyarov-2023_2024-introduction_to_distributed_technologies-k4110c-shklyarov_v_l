![image](https://github.com/user-attachments/assets/7f1c2e0d-5b3b-483c-8483-ec557b4a547a)Faculty: [FICT](https://fict.itmo.ru)<br>
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

Все прошло без ошибок, `configMap` создан.

####2. Создание `replicaSet` с 2 репликами контейнера.
####Передача переменных `REACT_APP_USERNAME`, `REACT_APPCOMPANY_NAME` через `configMap`.

В отличии от похожего объекта `deployment` из предыдущей лабораторной работы, `replicaSet` представляет собой более "низкий" уровень управления контейнерами. Он не поддерживает обновления, имеет более простую структуру, а его основной задачей является поддержка стабильного числа подов. `deployment` имеет более высокий уровень абстракции, он позволяет осуществлять управления обновлениями и оптимизацию жизненного цикла приложений. Кроме того, `replicaSet` создаются автоматически при создании `deployment`, и являются неотъемлимой его частью.

Для его создания также нужно написать манифест.
![image](https://github.com/user-attachments/assets/8c77b10f-2bf3-4e04-921c-511e05ab9d50)

Здесь мы указываем тип объекта `ReplicaSet`, и количество реплик =2. В спецификациях шаблона указываем имя контейнера и нунжый нам образ. Также в `envFrom: configMapRef` указываем имя нашего `congifMap`.

Применяем манифест и смотрим, какие создались поды.

```bash
kubectl apply -f replicaset.yaml
kubectl get replicaset
kubectl get pods
```

Смотрим на результат
![image](https://github.com/user-attachments/assets/e95e72df-ec95-447a-adbd-07553b5cc40b)

Здесь в колонке `STATUS` мы видим `CreateContainerConfigError`. Это означает что kubernetes не смог создать нужные контейнеры по указанным параметрам. Посмотрим логи одного из контейнеров, чтобы понять, в чем проблема.
```bash
kubectl describe pod react-app-replicaset-cfnj7
``` 
![image](https://github.com/user-attachments/assets/bc7e8ec9-8baf-4dba-b9e9-f4ca625e68ba)

Здесь сразу видна ошибка. Система не может найти указанный конфигурационный файл. Проблема в том, что в манифесте на создание replicaset имя configmap указано как "react-app-config", в то время как в манифесте конфигруаицонного файла он называется `react-config`. Чтобы исправить эту ошибку необходимо:
1. Удалить существующий объект replicaset.
2. Указать правильное имя в манифесте и повторить попытку.

![image](https://github.com/user-attachments/assets/4c0bc9cc-c0c6-4168-a0ce-38a06fcf01e0)

Теперь все верно. Оба контейнера успешно запущены и имеют сатус `Running`.
Последним шагом нужно проверить, что переменные окружения переданы верно. Для этого можно зайти в контейнер в интерактивном режиме командой `exec -it` и выполнить команду `printenv`.


####3. Включение аддона ingress и создание tsl сертификата.
Аддон ingress включается командой
```bash
minikube addons enable ingress
```
![image](https://github.com/user-attachments/assets/4c181d9c-b02f-4f6f-ba0d-80fd8549adea)

Теперь проверим статус дополнения:
```bash
kubectl get pods -n ingress-nginx
```
![image](https://github.com/user-attachments/assets/2b6f1e9e-cd6d-4986-b1c9-efa8df94b160)

Теперь можно переходить к созданию сертификата. Для этого потребовалось установить `openSSL` - ПО, которое позволяет использовать возможности шифрования для различных целей, включая защиту веб-серверов, шифрование файлов и др. Загружено с https://slproweb.com/products/Win32OpenSSL.html.

Далее необходимо было настроить переменные окружения и добавить путь для OpenSSL. Путь к программе выглядит так:
![image](https://github.com/user-attachments/assets/3c690b97-f185-4f6e-acc9-3853a9c04eab)

Проверим что все работает корректно, выведя версию в командной строке:
![image](https://github.com/user-attachments/assets/3b939c7d-f971-40ab-830c-09d7473a889c)



Контроллер запущен, все хорошо. Переходим к созданию сертификата.
