
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
2) Указать `lable` для запущеных ранее нод и назначить IP-адреса.
3) Разработать манифест для Calico.
4) Создать `deployment` с 2 репликами контейнера.
5) Создать сервис для доступа к подам.
6) Запустить режим проброса портов и подключиться к контейнерам через веб-браузер, проверить `Container name` и  `Container IP`.
7) Пропинговать поды.

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

Для создания IP-адресов первым делом необходимо удалить IP-пулы, созданные по умолчанию. Делается это при помощи команды:

```bash
minikube kubectl -- delete ippools default-ipv4-ippool
```
![image](https://github.com/user-attachments/assets/fbb926b8-6ece-4f59-96e5-fff52a07beb7)

Далее создаем 2 .yaml файла для конфигурации. Создавал при помощи редактора nano.

![image](https://github.com/user-attachments/assets/0138fcab-5144-4128-91c4-7cffe02f3e90)

![image](https://github.com/user-attachments/assets/ac85675c-f800-47a6-9906-239c836101a6)

Чтобы применить эти манифеста, сначала необходимо установить `calicoctl`

![image](https://github.com/user-attachments/assets/e7cba1eb-b6c1-430c-8656-cd26eedb2188)

Добавляет в $PATH
![image](https://github.com/user-attachments/assets/fecab717-de84-4635-87d5-161629f4a1aa)

Пробуем применить манифест с помощью `calicoctl`
![image](https://github.com/user-attachments/assets/a74497a5-5847-4219-96c1-187f6641c80d)

Видимо ошибку. Чтобы исправить попробуем вручную указать переменную KUBERNETES_MASTER:
![image](https://github.com/user-attachments/assets/c65aba19-aff3-4fff-b1de-0c3f51db81e4)

Снова пробуем применить манифест:
![image](https://github.com/user-attachments/assets/77201538-ba64-4328-9099-35100e8c1196)
IPPool успешно создан. Повторяем для второго файла
![image](https://github.com/user-attachments/assets/c0b7836c-321f-4f97-bf49-7551417c70a5)

Проверяем созданные пулы:
![image](https://github.com/user-attachments/assets/75a0eedb-8403-4dc4-8674-ba9722f4ba23)

### Создаyние `deployment` с 2 репликами контейнера

Создаем манифест с kind: deployment:
![image](https://github.com/user-attachments/assets/138d52cf-2f14-47d1-bad5-94ea2e897dad)
И применяем его:
![image](https://github.com/user-attachments/assets/8cb6e12d-1a34-466e-a900-fa4c1628e5d3)

Проверим состояние созданного объекта и подов:
![image](https://github.com/user-attachments/assets/2c308921-f6fc-4dc5-aaa2-3eb6e9ca8be1)

Все работает штатно

5) Создание сервиса
Создадим сервис для достуа к контейнерам при помощи команды и пробросим порты

```bash
minikube kubectl -- expose deployment cni-deployment --port=3000 --name=cni-service --type=ClusterIP
```
![image](https://github.com/user-attachments/assets/2fadfc0c-187b-4b76-99c0-2a8a31c3245d)


6. Проверяем отображение в браузере

Переходим в браузер по адресу localhost:3000
![image](https://github.com/user-attachments/assets/ef4b7c98-1f00-4c3e-adc0-1301e40fda74)

Поду пристаивается адрес, заданный заранее из пула. В данном случае мы видим, что поду присвоился адрес пода minikube-m02
![image](https://github.com/user-attachments/assets/cccc0684-ca69-430b-81bc-938c5ec8f44a)

7. Пингование

Сделаем запросы к каждому из подов при помощи коамнды PING. Для этого сначала подключимся к одному поду, а затем отпраим из него запрос к другому.

![image](https://github.com/user-attachments/assets/a618f681-700c-448a-b52e-799fb8c3d7ec)
![image](https://github.com/user-attachments/assets/f373fe72-2ee0-4beb-8d7a-c1357cc746a3)

Как мы видим, данные передается успешно без потерь.






