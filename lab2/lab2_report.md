![image](https://github.com/user-attachments/assets/5cd7e357-d54f-40d5-ac16-3996724bcce8)Faculty: [FICT](https://fict.itmo.ru)<br>
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)<br>
Year: 2024/2025<br>
Group: K4110c<br>
Author: Shklyarov Vladislav Leonidovich<br>
Lab: Lab2<br>
Date of create: 07.11.2024<br>
Date of finished: 14.09.2023<br>

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
![image](https://github.com/user-attachments/assets/58aeed34-6671-4b09-9516-1f15f285c5a2)


В этом манифесте есть несколько важных аспектов.
1) в параметре `kind` мы укаызваем `Deployment` - тип ресурса, который управляет развертыванием и обновлением набора контейнеров.
2) в `spec: replicas` указано значение `2` поскольку именно столько реплик контейнеров нам нужно поддерживать.
3) Появился параметр `selector:`. Он определяет, какие поды относятся к данному Deployment. В данном случае, этим Deployment будут управляться только поды, с соответсвующей меткой `app: frontend`.
4) Параметр `template` как раз описывает шаблон пода, который будет развернут данным `Deployment`. Именно здесь мы указвыаем нужную метку `app: frontend` по которой Kubernetes может находить и связывать поды.
5) В `spec` раздела `templates` указываем орбаз, а также две переменные окружения: `REACT_APP_NAME`, и `REACT_APP_COMPANY_NAME`.

Запустим манифест командой 

```bash
kubectl apply -f frontend-deployment.yaml
```
![image](https://github.com/user-attachments/assets/82734c28-acbf-439a-a820-c03ea95679e8)
Выводится сообщение о том, что Deployment успешно создан.
Проверим состояние объекта при помощи команды
```bash
kubectl get deployments
```
![image](https://github.com/user-attachments/assets/1065bfe2-7ab2-4a2f-83c4-fcbdf1c3983b)
Команда вывоидит на экран информацию о деплойментах. В данном случе 2 из 2 указанных в манифесте готовы к работе.

Также проверим поды, метка которых = frontend
```bash
kubectl get pods -l app=frontend
```
![image](https://github.com/user-attachments/assets/d51fced2-829c-4d64-95a5-a8bfd8a54f87)
Выводится информация о созаднных подах.

Прежде чем двигаться дальше проверим, как созданный Deployment справляется с восстановлением "сломанных" контейнеров. Для этого получим список подов и удалим один из них командой
```bash
kubectl delete pod frontend-deployment-f759496d5-4xl6f
```
![image](https://github.com/user-attachments/assets/c442726e-2ace-4c86-9cf7-76e402a5b20e)

Как видно на сркиншоте, когда под удалился, на его месте сразу же появился новый, с `-g52hm` на конце.

### 2) Создание сервиса для доступа к подам.
Для создания сервиса также можно создать манифест.
![image](https://github.com/user-attachments/assets/3997cad8-6af9-41dc-9e07-56a75c29f250)

В этом манифесте `kind: service` обозначает, что объект является сервисом. `targetPort: 3000` - Это порт внутри контейнера, где работает наше приложение. `port: 80` это порт на который направляется внешний тарфик. Тип сервиса `NodePort`.

Создадим сервис командой 
```bash
PS C:\Education\kubernetes_labs> kubectl apply -f frontend-service.yaml
```
И выведем его состояние 
```bash
PS C:\Education\kubernetes_labs> kubectl get svc frontend-service
```
![image](https://github.com/user-attachments/assets/37d94df0-b490-4bd3-80f4-acb8ff70f68c)

По аналогии с предыдущей работой мы можем узанть ip ноды кластера  `minikube ip -p minibox` и подключиться к ней напрямую. 

### 3) Пробрасываем порт

Выполним команду
```bash
kubectl port-forward svc/frontend-service 8082:80
```
 В результате приложение доступно на localhost:8082

 ![image](https://github.com/user-attachments/assets/b0b3fa75-b513-45ff-8b43-200941471375)
### 4) Проверка переменных
Обратим внимание на название контейнера: последние символы `-g52hm`. Попробуем удалить этот контейнер и посмотрим, как Kubernetes справится с этим.
![image](https://github.com/user-attachments/assets/e747a972-1fe3-4b60-b2da-4b19d99d28dc)
![image](https://github.com/user-attachments/assets/334d5f66-a1a7-4c0e-a09a-fcf0e89fda43)

Как видно, название контейнера изменилось на -2xkq4, в то время как переменные `REACT_APP_USERNAME` и `REACT_APP_COMPANY_NAME` остались прежними. Так происходит из-за того, что эти имена создаются на этапе инциализации объекта Deployment, и прописаны в манифесте. В результате они не изменяются даже при смене рабочего контейнера, для их изменения необходимо обновить манифест деплоймента и пересоздать поды с новыми значениями переменных. (более того, в задании они прописаны капсом, что намекает на то, что это -- константы, котоыре не должны меняться)

Что касается переменной `container_name`, то оно напрямую зависит от названия рабочего контейнера. Соответвенно, если у нас меняется контейнер - меняется и переменная.

### 5) Логи контейенеров 

Логи контейнеров доступны по следующей команде
```bash
kubectl logs <название контейнера>
```
![image](https://github.com/user-attachments/assets/c67de406-c4ec-4ac5-a054-4aa9e358c570)

### Схема:

![Схема](https://github.com/user-attachments/assets/78e4c172-9f7f-4fb1-8c86-b850bfc7ec6f)

