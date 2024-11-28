## Лабораторная работа №3 "Сертификаты и "секреты" в Minikube, безопасное хранение данных."
University: [ITMO University](https://itmo.ru/ru/) \
Faculty: [FICT](https://fict.itmo.ru) \
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies) \
Year: 2024/2025 \
Group: K4111c \
Author: Polenkov Vladislav \
Lab: Lab3 \
Date of create: 24.11.2024 \
Date of finished: 28.11.2024 \

### Описание
В данной лабораторной работе мы познакомимся с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube. 

### Цель работы
Познакомиться с сертификатами и "секретами" в Minikube, правилами безопасного хранения данных в Minikube. 

### Ход работы

- Необходимо создать `configMap` с переменными: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.

- Необходимо создать `replicaSet` с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и используя ранее созданный `configMap` передать переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` .

- Включить `minikube addons enable ingress` и сгенерировать TLS сертификат, импортировать сертификат в minikube. 

- Создать ingress в minikube, где указан ранее импортированный сертификат, FQDN по которому мы будем заходить и имя сервиса который мы создали ранее.

- В `hosts` прописать FQDN и IP адрес нашего ingress и попробовать перейти в браузере по FQDN имени. 

- Войти в веб приложение по своему FQDN используя HTTPS и проверить наличие сертификата.

### Результаты лабораторной работы

- Файлы с разработанными манифестами с расширением `.yaml`.

- Схема организации контейнеров и сервисов нарисованная в [draw.io](https://app.diagrams.net) или Visio.

- Скриншоты c результатами работы.

### Выполнение работы

#### Подключение Ingress

Подключим аддон ingress, с помощью команды:

```bash
minikube addons enable ingress
```

![image1](/lab3/images/image1.png)

#### Создание манифестов configMap, replicaSet и service

Ниже представлены манифесты [configMap.yaml](/lab3/configMap.yaml), [replicaSet.yaml](/lab3/replicaSet.yaml) и [service.yaml](/lab3/service.yaml).

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
data:
  REACT_APP_USERNAME: "sammyxd"
  REACT_APP_COMPANY_NAME: "sammyxd"
```
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: ifilyaninitmo/itdt-contained-frontend:master
          ports:
            - containerPort: 3000
              name: http
          envFrom:
            - configMapRef:
                name: config
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-lab3
spec:
  type: NodePort
  ports:
    - port: 3000
      protocol: TCP
      name: http
  selector:
    app: frontend
```
Применим их с помощью следующих команд:

```bash
minikube kubectl -- apply -f configMap.yaml
```

```bash
minikube kubectl -- apply -f replicaSet.yaml
```

```bash
minikube kubectl -- apply -f service.yaml
```

![image2](/lab3/images/image2.png)

#### Генерация сертификата

Сгенерируем сертификат следующей командой:

```bash
openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out selfsigned.crt -keyout selfsigned.key
```

![image3](/lab3/images/image3.png)

Далее создадим секрет с сертификатом:

```bash
minikube kubectl create secret tls secret-tls --key="selfsigned.key" --cert="selfsigned.crt"
```

![image4](/lab3/images/image4.png)

#### Создание манифеста ingress

Напишем манифест [ingress.yaml](/lab3/ingress.yaml) и применим его:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-lab3
spec:
  tls:
    - secretName: tls-secret
    - hosts:
        - lab3sammyxd.com
  rules:
    - host: lab3sammyxd.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: service-lab3
                port:
                  name: http
```

```bash
minikube kubectl -- apply -f ingress.yaml
```

![image5](/lab3/images/image5.png)

#### Запуск

Предварительно добавим в файл ```etc/hosts``` запись нашего домена:

![image6](/lab3/images/image6.png)

Запустим туннель:

```bash
minikube tunnel
```

![image7](/lab3/images/image7.png)

Далее перейдем по url: ```lab3sammyxd.com``` в браузере и проверим сертификат:

![image8](/lab3/images/image8.png)

![image9](/lab3/images/image9.png)

### Диаграмма организации

Схема организации кластера:

![image10](/lab3/images/image10.png)