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
5) Создать под и сервис для доступа к данному контейнеру;
6) Найти токен для доступа к vault и войти.

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

Информация о существующих кластерах 
```bash
minikube profile list
```
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


### 4. Manifest для развертывания пода с образом HashiCorp Vault.

Создаем файл vault-deployment.yaml

![image](https://github.com/user-attachments/assets/58d68d80-2ab6-4269-99d5-2c88336175c8)

В этом файле указывается:
1) apiVersion: указывает версию API Kubernetes, которую мы используем для создания объекта. Для базовых объектов, таких как Pod, подходит версия v1.
2) Kind: определяется тип создаваемого объекта. Мы создаем объект типа Pod - минимальную сущность в Kubernetes, с которой можно осуществлять полноценную работу. Она может содержать один и более контейнеров.
3) metaData: раздел с метаданными, в котором содержится информация для идентификации и организации объекта. В данном случае прописаны имя vault и лэйбл с меткой app. Не обязательный параметр, но он упрощает идентификацию и организацию объектов. Например метка позволяет группировать объекты, использовать их для фильтрации и связывания, масштабирования и тд.
4) sepc: отвечает за спецификации пода, включая его контейнеры и настройки. В containers указаны имя и образ контейнера, а также его порты (в данном случае единственный порт 8200).

### 5. Создание пода и сервиса для доступа к контейнеру.

Здесь необходимо запустить Pod с помощью манифеста. Для этого выполняем следующую команду:

```bash
minikube kubectl -- apply -f vault-deployment.yaml --validate=false

```
На всякий случай прописал флаг --validate=false чтобы исключить строгую проверку манифеста воизбежание ошибок. Проверяем статус созданного пода командой
```bash
minikube kubectl -- get pod vault -o wide

```
![image](https://github.com/user-attachments/assets/76a88b6b-0f7a-4e97-9985-bd51f2f95aeb)

Теперь создаем сервис при помощи команды
```bash
minikube kubectl -- expose pod vault --type=NodePort --port=8200
```
Указываем порт 8200.
Проверяем что сервер создался командой 
```bash
minikube kubectl -- get svc
```
![image](https://github.com/user-attachments/assets/5c36a29b-2126-45a2-a613-e79358e3e6f1)

Здесь мы видим что сервер создан. Также мы видим информацию о портах. 8200 - это порт внутри контейнера, который мы указывали в манифесте. Вторая часть, после двоеточия, 31781 это порт, на который Minikube пробрасывает порт контейнар "во внешний мир". Далее есть два варианта.

#### Вариант 1. Подключение через внешний IP и порт, проброшенный Minikube.

Для использования этого способа нам необходимо знать внешний IP адрес нашей Minikube VM. Узнать его можно при помощи команды

```bash
minikube ip
```
![image](https://github.com/user-attachments/assets/35f93900-b717-4f7a-b171-4f7959926ca3)

Как мы видим, IP адрес = 192.168.59.102
Далее нам остается указать этот адрес в адресной строке и добавить к нему назначенный порт

```bash
http://192.168.59.102:31781/
```
В результате мы перейдем на страницу Vault
![image](https://github.com/user-attachments/assets/a03f901f-5571-4f1c-883d-7b43253cf1d2)

#### Вариант 2. Проброс порта на локальную машину.
Для этого способа используем команду 

```bash
minikube kubectl -- port-forward service/vault 8200:8200
```
После этого веб-интерфейс становится доступен по ссылке
```bash
http://localhost:8200/
```
![image](https://github.com/user-attachments/assets/1be8cd86-41dd-49ac-ba42-b44f97c94198)

### 6. Ищем токин и входим в Vault.

Для поиска токена необходимо просмотреть логи контейнера при помощи команды

```bash
minikube kubectl -- port-forward service/vault 8200:8200
```
![image](https://github.com/user-attachments/assets/2cafce0e-0345-439e-a9ac-2679095f30b8)

Среди прочей информации здесь указан root token для доступа.
Вводим его в поле и осущетсвляет вход.
![image](https://github.com/user-attachments/assets/56810da9-fb37-4f5c-979c-ff66260249fe)

Аналогично работает и по ссылке 

```bash
http://192.168.59.102:31781/
```

![image](https://github.com/user-attachments/assets/b734eadd-ceae-4614-b38d-7ab5380cba05)


------
