
Faculty: [FICT](https://fict.itmo.ru)<br>
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)<br>
Year: 2024/2025<br>
Group: K4110c<br>
Author: Shklyarov Vladislav Leonidovich<br>
Lab: Lab3<br>
Date of create: 06.11.2024<br>
Date of finished: 14.11.2024<br>

### Цель работы
Познакомиться с CNI Calico и функцией IPAM Plugin, изучить особенности работы CNI и CoreDNS.

### Ход работы
1) Запустить minikube с плагином `CNI=Calico` и проверить.
2) Указать `lable` для запущеных ранее нод.
3) Разработать манифест для Calico.
4) Создать `deployment` с 2 репликами контейнера.
6) Создать сервис для доступа к подам.
7) Запустить режим проброса портов и подключиться к контейнерам через веб-браузер, проверить `Container name` и  `Container IP`.
8) Пропинговать поды.

### Выполнение работы
#### 1. Запуск Minikube с плагином `CNI=Calico`.

Запускаем кластер minikube при помощи команды

```bash
minikube start --nodes=2 --network-plugin-cni --cni=caliico
```
![image](https://github.com/user-attachments/assets/5c0f18f5-3162-4306-8c93-d6ccecfc94d5)

Проверяем созданные ноды при помощи:
```bash
minikube kubectl -- get nodes
```
![image](https://github.com/user-attachments/assets/64743cd0-a54d-4a8a-bdec-21f19027a02b)

Также можем проверить узлы calico


Они тоже работают исправно.

#### 1. Указываем lables для контейнеров

Для пометки узлов присваиваем каждому из них label, например, по географическому расположению:

```bash
minikube kubectl -- labels nodes minikube-m02 zone=south
minikube kubectl -- labels nodes minikube zone=north
```

![image](https://github.com/user-attachments/assets/b286b529-f789-498b-8df6-d08df744d2ca)

При выозове описания нода мы видим, что наше значение сохранилось среди лейблов.

![image](https://github.com/user-attachments/assets/2fdea339-d546-4480-b8c9-b943be3b6d01)
![image](https://github.com/user-attachments/assets/34ac7347-1f68-407b-bd05-b4736127584c)


