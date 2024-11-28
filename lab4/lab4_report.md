## Лабораторная работа №4 "Сети связи в Minikube, CNI и CoreDNS"
University: [ITMO University](https://itmo.ru/ru/) \
Faculty: [FICT](https://fict.itmo.ru) \
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies) \
Year: 2024/2025 \
Group: K4111c \
Author: Polenkov Vladislav \
Lab: Lab4 \
Date of create: 25.11.2024 \
Date of finished: 28.11.2024 \

### Описание

Мы познакомимся с сетями связи в Minikube. Особенность Kubernetes заключается в том, что у него одновременно работают `underlay` и `overlay`  сети, а управление может быть организованно различными CNI.

### Цель работы

Познакомиться с CNI Calico и функцией `IPAM Plugin`, изучить особенности работы CNI и CoreDNS.

### Ход работы

- При запуске minikube установить плагин `CNI=calico` и режим работы `Multi-Node Clusters` одновеременно, в рамках данной лабораторной работы нужно развернуть 2 ноды.

- Проверить работу CNI плагина Calico и количество нод.

- Для проверки работы Calico попробовать одну из функций под названием `IPAM Plugin`.

- Для проверки режима `IPAM` необходимо для запущеных ранее нод указать `label` по признаку стойки.
  
- После этого необходимо разработать манифест для Calico который бы на основе ранее указанных меток назначал бы IP адреса "подам" исходя из пулов IP адресов которые указаны в манифесте.

- Необходимо создать `deployment` с 2 репликами контейнера [ifilyaninitmo/itdt-contained-frontend:master](https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontend) и передать переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.

- Создать сервис через который будет доступ на эти "поды". 

- Запустить в `minikube` режим проброса портов и подключитесь к вашим контейнерам через веб браузер.

- Проверьте на странице в веб браузере переменные `Container name` и `Container IP`. Изменяются ли они? Если да то почему?

- Используя `kubectl exec` зайдите в любой "под" и попробуйте попинговать "поды" используя `FQDN` имя соседенего "пода", результаты пингов необходимо приложить к отчету.  


### Результаты лабораторной работы

- Файлы с разработанными манифестами с расширением `.yaml`.

- Схема организации контейнеров и сервисов нарисованная в [draw.io](https://app.diagrams.net) или Visio.

- Скриншоты c результатами работы.

### Выполнение работы

#### Настройка среды

Установим ```calico``` и режим работы ```multinode```. После этого проверим создание двух нод.

```bash
minikube start --network-plugin=cni --cni=calico --nodes 2 -p multinode
```

![image1](/lab4/images/image1.png)

```bash
kubectl get nodes
```

![image2](/lab4/images/image2.png)

Также проверим количество подов:

```bash
kubectl get pods -l k8s-app=calico-node -A
```

![image3](/lab4/images/image3.png)

Далее, каждую ноду пометим по признаку стойки.

```bash
kubectl label nodes multinode-demo ra=mark1 --overwrite
kubectl label nodes multinode-demo-m02 ra=mark2 --overwrite
```
![image4](/lab4/images/image4.png)

Получим у ```calico``` стандартный ```ip-pool```:

```bash
calicoctl get ippool -o wide --allow-version-mismatch
```

![image5](/lab4/images/image5.png)

```bash
calicoctl delete ippools default-ipv4-ippool --allow-version-mismatch
```

![image6](/lab4/images/image6.png)

Далее подключим нашу конфигурацию [calico.yaml](/lab4/calico.yaml):

```yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: ra-mark1-pool
spec:
  cidr: 192.168.0.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: ra == "mark1"

---

apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: ra-mark2-pool
spec:
  cidr: 192.168.1.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: ra == "mark2"
```

```bash
calicoctl apply -f calico.yaml --allow-version-mismatch
```

![image7](/lab4/images/image7.png)

Проверим создание пулов:

```bash
calicoctl get ippool -o wide --allow-version-mismatch
```

![image8](/lab4/images/image8.png)

Далее применим манифесты [deployment](/lab4/frontend-deployment.yaml) и [service](/lab4/frontend-service.yaml) из второй лабораторной работы.


```bash
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
```

![image9](/lab4/images/image9.png)

Далее пробросим порты:

```bash
kubectl port-forward service/frontend-deployment 3000:3000
```

![image10](/lab4/images/image10.png)

#### Запуск

Теперь можно перейти на ```localhost:3000```

![image11](/lab4/images/image11.png)

Далее получим все ip-адреса контейнеров ```calico``` и пропингуем поды.

```bash
kubectl describe pods | grep IP
```

![image12](/lab4/images/image12.png)

```bash
kubectl exec -ti frontend-deployment-76dcbb964d-7hcgt -- sh
```

![image13](/lab4/images/image13.png)

### Диаграмма организации

Схема организации кластера:

![image14](/lab4/images/image14.png)