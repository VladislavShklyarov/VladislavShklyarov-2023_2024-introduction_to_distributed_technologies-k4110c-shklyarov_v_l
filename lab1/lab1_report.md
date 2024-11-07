![image](https://github.com/user-attachments/assets/96da9408-1a36-46d6-98b2-741a4f002f3d)University: [ITMO University](https://itmo.ru/ru/)<br>
Faculty: [FICT](https://fict.itmo.ru)<br>
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)<br>
Year: 2024/2025<br>
Group: K4110c<br>
Author: Shklyarov Vladislav Leonidovich<br>
Lab: Lab1<br>
Date of create: 06.11.2024<br>
Date of finished: 31.09.2023<br>


### Цель работы
Ознакомиться с инструментами Minikube и Docker, развернуть свой первый "под".

### Ход работы
1) Установить Docker,  Minikube, kubectl, Virtual Machine;
2) Развернуть minikube cluster;
3) Скачать образ HashiCorp Vault;
4) Написать manifest для развертывания пода с образом HashiCorp Vault;
5) Создать сервис для доступа к данному контейнеру;
6) Прокинуть порт для доступа к контейнеру;
7) Найти токен для доступа к vault и войти.

### Выполнение работы
#### 1. Установка Docker, Minikube, kubectl, Oracle Virtual Box.

Docker Desktop необходим для управления контейнерами и создания образов. Скачал с официального сайта https://docs.docker.com/get-started/get-docker/. Убедился, что Docker работает. ![image](https://github.com/user-attachments/assets/91c8aaae-60aa-407d-b3ec-b4b652654c5f)
 

Minikube для создания локального кластера Kubernetes, что нужно для разработки и тестирования. Скачал с сайта https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download и проверил его работу командой

```bash
minikube version.
```



![image](https://github.com/user-attachments/assets/a4a17a75-fa3e-4377-be24-04411e791077)



kubectl необходим для управления кластером Kubernetes и развертывания приложений. Аналогично убедился в установке с помощью 

```bash
kubectl version --client.
```

![image](https://github.com/user-attachments/assets/87470bcb-03d7-469d-a0e6-2181b75665e5)
```bash

```

Виртуальная машина: Использовал Oracle Virtual Box для виртуализации, так как Minikube требует виртуальной среды. Скачал с официального сайта https://www.virtualbox.org/wiki/Downloads

### 2. Раворачивание minikube cluster
Развернул первый кластер minikube при помощи команды minikube start --driver=virtualbox.
![image](https://github.com/user-attachments/assets/db589ab2-6b35-4f2a-b869-e6358835babd)

Проверил статус кластера командой
```bash
minikube status
```


![image](https://github.com/user-attachments/assets/c3a97348-6cdc-49a6-8af0-322470587c74)


Для тренировки запустил еще один кластер под названием minibox, указав параметры: колиство ядер = 2, среда выполнения = containerd(вместо docker), оперативная память = 2gb, разамер диска = 10gb.
![image](https://github.com/user-attachments/assets/087b8870-d1d7-4b72-bf47-b632eb711792)

Информация о существующих кластерах (команда minikube profile list)
![image](https://github.com/user-attachments/assets/ec486af7-be29-4354-939c-c7704afd6cca)

### 3. Скачивание образа HashiCorp Vault.

Cкачал образ при помощи команды 
```bash
docker pull vault:1.13.3
```
Проверил существубщие образы командой

```bash
docker images
```

![image](https://github.com/user-attachments/assets/db013f34-6a61-439f-a5b8-d404e23b1099)





- Файл с разработанным вами манифестом для развертывания "пода" с расширением `.yaml`.

- Схема организации контейеров и сервисов нарисованная вами в [draw.io](https://app.diagrams.net) или Visio.

- Ответы на вопросы (по возможности), скриншоты c результатами работы.


------
