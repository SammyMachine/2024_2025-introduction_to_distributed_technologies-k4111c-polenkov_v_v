## Лабораторная работа №1 "Установка Docker и Minikube, мой первый манифест."
University: [ITMO University](https://itmo.ru/ru/) \
Faculty: [FICT](https://fict.itmo.ru) \
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies) \
Year: 2024/2025 \
Group: K4111c \
Author: Polenkov Vladislav \
Lab: Lab1 \
Date of create: 14.11.2024 \
Date of finished: ...

### Описание
Это первая лабораторная работа в которой мы сможем протестировать Docker, установить Minikube и развернуть свой первый "под".

### Цель работы
Ознакомиться с инструментами Minikube и Docker, развернуть свой первый "под".

### Ход работы
- Установить Docker и Minikube; 
- Развернуть minikube cluster;
- Скачать образ HashiCorp Vault;
- Написать манифест для развертывания пода с образом HashiCorp Vault;
- Создать сервис и прокинуть порт для доступа к контейнеру;
- Найти токен для доступа к vault;
- Войти с помощью токена;

### Выполнение работы

#### Установка Docker и Minikube

Выполним установку Docker и Minikube согласно инструкции, удостоверимся, что они действительно установились.

![image1](/lab1/images/image1.png)

Как видим, все успешно установлено.

#### Развертывание minikube cluster

Запустим minikube cluster при помощи команды ```minikube start```:

![image2](/lab1/images/image2.png)


#### Загрузка образа HashiCorp Vault

Далее необходимо скачать образ HashiCorp Vault, сделаем это через интерфейс docker. Убедимся, что образ действительно скачан:

![image3](/lab1/images/image3.png)

#### Манифест для развертывания пода

Был написан манифест [vault-pod.yaml](/lab1/vault-pod.yaml) для создания пода с образом HashiCorp Vault:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vault
  labels:
    app: vault
spec:
  containers:
  - name: vault
    image: vault:1.13.3
    ports:
    - containerPort: 8200
```
- **apiVersion** — версия API Kubernetes.

- **kind** — тип ресурса, который мы создаем в Kubernetes (`Pod`).

- **metadata** — метаданные для ресурса:
  - **name** — уникальное имя для этого пода (`vault`), чтобы его можно было идентифицировать.
  - **labels** — набор меток, которые помогают группировать и искать ресурсы:
    - **app** — метка с названием `vault`, которая позволяет легко найти все поды, связанные с этим приложением.

- **spec** — спецификация, описывающая, как должен быть развернут под:
  - **containers** — список контейнеров, которые нужно запустить внутри пода (в данном случае это один контейнер).
    - **name** — имя контейнера (`vault`).
    - **image** — Docker-образ, который будет использоваться (`vault:1.13.3`).
    - **ports** — список портов, на которых контейнер будет слушать входящие соединения:
      - **containerPort** — порт внутри контейнера, на котором работает приложение (`8200`).

#### Создание сервиса для доступа контейнеру

Применим манифест пода vault в кластере c помощью команды:

```bash
minikube kubectl -- apply -f vault-pod.yaml
```

![image4](/lab1/images/image4.png)

С помощью следующей команды создадим сервис c перенаправлением трафика на под vault через порт 8200.

```bash
minikube kubectl -- expose pod vault --type=NodePort --port=8200
```

С помощью команды ниже, проверим, что сервис создан:

```bash
kubectl get service
```

 Результат выполнения команд:

![image5](/lab1/images/image5.png)

#### Прокидывание порта

Далее необходимо установить соединение между портом 8200 на машине и портом 8200 сервиса vault. Это можно сделать с помощью команды ниже:

```bash
minikube kubectl -- port-forward service/vault 8200:8200
```

Результат выполнения:

![image6](/lab1/images/image6.png)

Перейдем на `localhost:8200`:

![image7](/lab1/images/image7.png)

#### Поиск токена

С помощью команды ниже, посмотрим логи и найдем токен доступа в vault:

```bash
minikube kubectl logs vault
```

В логах ниже содержится токен:

![image8](/lab1/images/image8.png)

#### Вход в хранилище

Войдем в хранилище с помощью полученного токена:

![image9](/lab1/images/image9.png)

![image10](/lab1/images/image10.png)

### Диаграмма организации

Схема организации кластера:

![image11](/lab1/images/image11.png)