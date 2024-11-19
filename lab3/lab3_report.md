![image](https://github.com/user-attachments/assets/d9ae607d-c4be-419f-ae9d-78a9f9f1300e)![image](https://github.com/user-attachments/assets/f8f81c21-d800-4486-ace0-caee7e2fa8e5)![image](https://github.com/user-attachments/assets/7f1c2e0d-5b3b-483c-8483-ec557b4a547a)Faculty: [FICT](https://fict.itmo.ru)<br>
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

Следующим шагом к созданию сертификата является создание приватного ключа. Он используется для шифрования данных, подписи сертификатов и другий криптографических операций. Выполняется командой для генерации приватного ключа, с указанием алгоритма - RSA.

```bash
openssl genpkey -algorithm RSA -out private.key
```
Результатом является последовательность символов:

![image](https://github.com/user-attachments/assets/bb25b0c4-ee3a-45b4-866b-4c3c06d20fde)

Далее нужно сформировать запрос на подписание сертификата, с явным указанием пути к файлу "openssl.cnf" (без явного указания программа пыталась найти этот файл в несуществующей директории).

```bash
openssl req -new -key private.key -out csr.pem -config "C:\Program Files\OpenSSL-Win64\bin\cnf\openssl.cnf"
```
После выполнения этой команды программа попросила ввести информацию, включенную в сертификат: страну, город, организацию и тд.

![image](https://github.com/user-attachments/assets/d87e9089-7e35-485a-b08f-e103e92431dc)

После этого можно проверить, что запрос на формирование сертификата выполнился успешно.

```bash
cat csr.pem
```
![image](https://github.com/user-attachments/assets/4932db7b-23ca-430a-b4bf-7b59036dbb78)

Наконец, можно создать сертификат.

```bash
openssl req -x509 -days 365 -key private.key -in csr.pem -out certificate.crt -config "C:\Program Files\OpenSSL-Win64\bin\cnf\openssl.cnf"
```

Здесь указвается количество дней, которое действителен сертификат `-days 356`, файл с приватным ключом `-key private.key`, файл с запросом на формирование ключа `-in csr.pem` и само название сертификата `-out certificate.crt`. Здесь также пришлось указать явный путь к файлу "openssl.cnf".

Файл "certificate.crt" создался в рабочей дериктории. 
![image](https://github.com/user-attachments/assets/50eb688f-6d41-4ccb-bb78-4de6c3c88477)

Следующим шагом является импорт сертификата в minikube. Для хранения сертификатов kubernetes использует секреты. Для его создания необходимо выполнить следующую команду:

```bash
kubectl create secret tls my-app-tls --key private.key --cert certificate.crt
```
Здесь указывются названия файлов с ключом и сертификатом а также название секрета, в данном случае `my-app-tls`.

![image](https://github.com/user-attachments/assets/490eb7de-09ca-43ff-9c68-c8b34824db49)

Секрет создан.

####4) Создание ingress.
Теперь можно переходить к созданию ingress манифеста для маршрутизации приложения через доменное имя.

![image](https://github.com/user-attachments/assets/69b80c2c-7b25-4780-bf25-0bedb14122fc)


Основные моменты: `secretName: my-app-tls` - тот самый секрет, созданный на предыдущем шаге. `host: react-app.local` - доменное имя, указанное при создании серфитиката. `service.name: react-app-service`: указываем имя сервиса (предстоит создать).

Теперь создаем манифест для сервиса.

![image](https://github.com/user-attachments/assets/f0cabdc8-9c63-403f-83b6-eb619533f163)

Приминяем манифесты

```bash
kubectl apply -f ingress.yaml
kubectl apply -f service.yaml  # если сервис еще не создан
```

Результат:

![image](https://github.com/user-attachments/assets/bd911b88-a94a-43d7-8b36-336dded824bd)

####5) Прописываем FQDN и IP адресс ingress в `hosts` и переходим в браузер.

Чтобы прописать параметры в `hosts` необходимо открыть файл `C:\Windows\System32\drivers\etc\hosts` от имени администратора и внести туда следующие изменения:






