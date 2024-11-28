## Лабораторная работа №2 "Развертывание веб сервиса в Minikube, доступ к веб интерфейсу сервиса. Мониторинг сервиса."
University: [ITMO University](https://itmo.ru/ru/) \
Faculty: [FICT](https://fict.itmo.ru) \
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies) \
Year: 2024/2025 \
Group: K4111c \
Author: Polenkov Vladislav \
Lab: Lab2 \
Date of create: 15.11.2024 \
Date of finished: 28.11.2024 \

### Описание
В данной лабораторной работе мы познакомимся с развертыванием полноценного веб сервиса с несколькими репликами.

### Цель работы
Ознакомиться с типами "контроллеров" развертывания контейнеров, ознакомится с сетевыми сервисами и развернуть свое веб приложение.

### Ход работы
- Необходимо создать `deployment` с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и передать переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.

- Создать сервис через который будет доступ на эти "поды". 

- Запустить в `minikube` режим проброса портов и подключиться к своим контейнерам через веб браузер.

- Проверить на странице в веб браузере переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` и `Container name`. Изменяются ли они? Если да то почему?

- Проверить логи контейнеров, приложить логи в отчёт.

### Результаты лабораторной работы

- Файлы с разработанными манифестами с расширением `.yaml`.

- Схема организации контейнеров и сервисов нарисованная в [draw.io](https://app.diagrams.net) или Visio.

- Скриншоты c результатами работы.

### Выполнение работы

#### Создание Deployment

Опишем наш deployment с помощью манифеста [frontend-deployment.yaml](/lab2/frontend-deployment.yaml):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  labels:
    app: frontend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend-deployment
  template:
    metadata:
      labels:
        app: frontend-deployment
    spec:
      containers:
        - name: frontend-deployment
          image: ifilyaninitmo/itdt-contained-frontend:master
          env:
            - name: REACT_APP_USERNAME
              value: "sammyxd"
            - name: REACT_APP_COMPANY_NAME
              value: "sammyxd"
          ports:
            - containerPort: 3000
```

По сравнению с прошлой работой, структура манифеста притерпела изменения. 

Поменялся **kind** с `Pod` на `Deployment` и теперь в **spec** заполняются следующие поля:
- **replicas** — указывает количество реплик контейнеров, которые нужно создать.

- **selector** — определяет группу реплик, которую мы создаем с помощью `Deployment` и описывает, какими подами она может управлять. 

- **template** — описывает шаблон для создаваемых подов с их конфигурацией.

- **env** — задает переменные окружения внутри контейнеров(`REACT_APP_COMPANY_NAME` и `REACT_APP_USERNAME`).

#### Создание сервиса

Напишем манифест [frontend-service.yaml](/lab2/frontend-service.yaml) для создания сервиса:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-deployment
spec:
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      protocol: TCP
      name: http
  selector:
    app: frontend-deployment
```
- **port** — это порт, открытый на уровне Kubernetes, через который сервис доступен для внешних запросов или взаимодействия с другими сервисами внутри кластера.

- **targetPort** — это целевой порт, на который перенаправляются запросы внутри контейнера. Например, запросы, поступающие на `port` сервиса (`3000`), будут направляться на тот же порт 3000 внутри контейнера.

#### Запуск

Создадим деплоймент и сервис при помощи команд:

```bash
minikube kubectl -- apply -f frontend-deployment.yaml
```

```bash
minikube kubectl -- apply -f frontend-service.yaml
```

![image1](/lab2/images/image1.png)

Проверим создание с помощью команд:

```bash
kubectl get deployments
```

```bash
kubectl get pods
```

```bash
kubectl get services
```

![image2](/lab2/images/image2.png)

Далее пробросим порты и зайдем в браузер:

```bash
kubectl port-forward service/frontend-deployment 3000:3000
```

![image3](/lab2/images/image3.png)

С помощью следующей команды, узнаем URL, на котором работает сервис:

```bash
minikube service frontend-deployment
```

![image4](/lab2/images/image4.png)

![image5](/lab2/images/image5.png)

#### Логи

![image6](/lab2/images/image6.png)

### Диаграмма организации

Схема организации кластера:

![image7](/lab2/images/image7.png)