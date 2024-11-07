![image](https://github.com/user-attachments/assets/5cd7e357-d54f-40d5-ac16-3996724bcce8)Faculty: [FICT](https://fict.itmo.ru)<br>
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)<br>
Year: 2024/2025<br>
Group: K4110c<br>
Author: Shklyarov Vladislav Leonidovich<br>
Lab: Lab2<br>
Date of create: 07.11.2024<br>
Date of finished: 31.09.2023<br>

### Цель работы

Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение. 

### Ход работы
1) Создать `deployment` с 2 репликами контейнера.
2) Создать сервис для доступа к подам.
3) Запустить в `minikube` режим проброса портов и подклюиться к контейнерам через веб-браузер. 
4) Проверка на странице в веб браузере переменных `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` и `Container name`. 
5) Проверка логов для контейнеров.

### Выполнение работы

#### 1. Создание `deployment` и 2 реплик контейнера.

В первую очередь скачиваем образ для контейнеров:
```bash
docker pull ifilyaninitmo/itdt-contained-frontend:master
```
![image](https://github.com/user-attachments/assets/8c11edf0-e508-4cf1-af77-8026cb0b708f)


Проверяем список контейнеров:
```bash
docker images
```

![image](https://github.com/user-attachments/assets/9681b0f4-977d-45e5-be04-7dea28432e19)

Вместо стандартного кластера `minikube`, в котором мы работаем по умолчанию, я хочу переключиться на кластер `minibox`. Для этого
1. Посмотрим текущую конфигурацию при помощи команды
```bash
kubectl config current-context
```
![image](https://github.com/user-attachments/assets/7dc35562-c68a-4a83-b068-98678ad93db6)

В ней указан `minikube`. Переключимся на кластер `minibox` и снова проверим конфигурацию
```bash
kubectl config use-context minibox
kubectl config current-context
```
![image](https://github.com/user-attachments/assets/1f58f07c-f9df-410a-8194-9830d85d697e)

Сработало, теперь мы находимся в кластере `minibox`.

Теперь можно приступать к написанию манифеста для создания `deployment`.

В этом манифесте есть несколько важных аспектов.
1) в параметре `kind` мы укаызваем `Deployment` - тип ресурса, который управляет развертыванием и обновлением набора контейнеров.
2) в `spec: replicas` указано значение `2` поскольку именно столько реплик контейнеров нам нужно поддерживать.
3) Появился параметр `selector:`. Он определяет, какие поды относятся к данному Deployment. В данном случае, этим Deployment будут управляться только поды, с соответсвующей меткой `app: frontend`.
4) Параметр `template` как раз описывает шаблон пода, который будет развернут данным `Deployment`. Именно здесь мы указвыаем нужную метку `app: frontend` по которой Kubernetes может находить и связывать поды.



