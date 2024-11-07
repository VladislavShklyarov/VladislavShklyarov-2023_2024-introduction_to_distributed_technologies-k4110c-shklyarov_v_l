Faculty: [FICT](https://fict.itmo.ru)<br>
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
docker pull ifilyaninitmo/itdt-contained-frontend
```







