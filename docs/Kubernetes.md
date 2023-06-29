# Kubernetes

# Содержание

* [Начало работы](#Начало-работы)
> * [](#)
> * [](#)
> * [](#)
> * [](#)
> * [](#)
> * [](#)
> * [](#)

# Начало работы

## Установка


# Словарь

Основной компонент Kubernetes это Cluster.

Создаётся Kubernetes Cluster состоящий из Nodes.

Nodes существует два типа:
* Master Node - сервер, который управляет Worker Nodes.
* Worker Node - сервер, на котором запускаются и работают контейнеры.

Master Node - сервер, на котором работают 3 главных процесса Kubernetes.

* kube-apiserver
* kube-controller-manager
* kube-sheduler

Worker Node - сервер, на котором работают два главных процесса Kubernetes.

* kubelet
* kube-proxy

* Pod – объект в котором работают один или больше Docker контейнеров.
* Deployment – сэт одинаковых подов, нужен для Auto scaling и для обновления Docker image,
держит минимальное количество работающих подов.
* Service – предоставляет доступ к Deployment через:
ClusterIP, NodePort, LoadBalance или ExternalName.
* Nodes – сервера где все это работает.
* Cluster – логическое объединение нодов.


# Базовые объекты

## Начальная конфигурация

[Наверх](#содержание)

Любая конфигурация файла Kubernetes содержит в себе:

* `apiVersion` - версия API Kubernetes для данного объекта.
* `kind` - указывает тип объекта (Pod, Deployment, Service и т.д.).
* `metadata` – метаданные, которые включают имя (name), пространство имен (namespace),
метки (labels) и аннотации (annotations) и другую информацию о модуле.


## Pod

[Наверх](#содержание)

Pod - это минимальная единица для управления Kubernetes.
Pod - это сетевая абстракция, это один или несколько контейнеров,
предназначенных для определённой задачи.

Пример pod.yml.

    apiVersion: v1                      # версия
    kind: Pod                           # тип объекта
    metadata:
      name: my-pod                      # имя pod'a
      namespace: namespace-web          # указание namespace

    spec:                               # описание как должен выглядеть pod
      containers:
      - name: nginx-server
        image: nginx:lts-alpine

      ports:                            # Описание портов
      - name: nginx-server
        containterPort: 80
        protocol: TCP

Команда для запуска.

    kubectl apply -f pod.yml

Удаление.

    kubectl delete -f pod.yml

Проверка живучестви pod:

1) проверка HTTP GET - выполняет запрос HTTP GET на IP-адрес, порт и
путь контейнера, которые будут указаны.
2) проверка сокета TCP - пытается открыть TCP-подключение к указанному
порту контейнера. Если подключение установлено успешно, то проверка сработала.
3) проверка Exec - выполняет произвольную команду внутри контейнера и
проверяет код состояния на выходе из команды.

    apiVersion: v1
    kind: pod
    metadata:
      name: kubia-liveness
    spec:
      containers:
      – image: luksa/kubia-unhealthy
        name: kubia
        livenessProbe:
          httpGet:
            path: /
            port: 8080


## Namespace

[Наверх](#содержание)

`Namespace` абстракция, которая позволяет разградить
разные приложения между различными пользователями.

Пример файла namespace.yaml.

    apiVersion: v1
    kind: Namespace
    metadata:
      name: <namespace-name>

Создание namespace.

    kubectl apply -f namespace.yaml

Удаление namespace.

    kubectl delete -f namespace.yaml

Получение всех объектов у определённого namespace.

    kubectl get all --namespace <namespace-name>


## Service

[Наверх](#содержание)

Service - объект, который определяет логический набор подов и политику доступа к ним.

Пример service.yml.

    apiVersion: v1
    kind: Service
    metadata:
      namespace: namespace-web
      name: service-name
    spec:
      selector:
        app: nginx-server
      ports:
        - protocol: TCP
          port: 80
          targetPort: 8080


Создание service.

    kubectl apply -f service.yml


## ReplicaSet

[Наверх](#содержание)

Занимается тем в каком количестве приложение должно запускаться.

label - метки.
selector -



## Deployment

[Наверх](#содержание)

нужен для обновления версии приложения.

*Пример:*

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      namespace: namespace-name
      labels:
        app: nginx-server
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx-server
      template:
        metadata:
          labels:
            app: nginx-server
        spec:
          containers:
          - name: web
            image: nginx:latest
            ports:
            - containerPort: 80

Обновление новых изменений.

    strategy:
       type: RollingUpdate
       rollingUpdate:
         maxUnavailable: 1


Probes

Liveness probe
* Контроль за состоянием приложения во время его жизни.
* Исполняется постоянно


Readiness probe
* Проверяет, готово ли приложение принимать трафик.
* В случае неудачного выполнения, приложение убирается из балансировки.
* Испольляется постоянно.


Resources

Limits
* Количество ресурсов, которые POD может использовать
* Верхняя граница

Requests
* Количество ресурсов которые резервируются для PODа на ноде
* Не делятся с другими PODами на ноде


## Secret

[Наверх](#содержание)

generic - пароли / токены для приложений.
docker-registry - данные для авторизации в docker registry.
tls - TLS сертификаты, для Ingress.

Шаблон.

    kubectl create secret <type> <name> --from-literal=<key>=<value>

Пример.

    kubectl create secret generic test --from-literal=test1=qwerty

Просмотр секрета.

    kubectl get secret <name> -o yaml


## DaemonSet

[Наверх](#содержание)


## ConfigMap

[Наверх](#содержание)

# ReplicationController

[Наверх](#содержание)

обеспечивает поддержание постоянной работы его модулей. Если модуль
исчезает по любой причине, например в случае исчезновения узла из кластера или потому,
что модуль был вытеснен из узла, контроллер репликации
замечает отсутствующий модуль и создает сменный модуль.


# Команды

kubectl get <type>

Получение pods.

    kubectl get pod
    kubectl get po
    kubectl get pod --show-labels

Получить информацию в виде yaml.

    kubectl get po <name-pod> -o yaml

Получение ReplicaSet.
    kubectl get replicaset
    kubectl get rs



kubectl create <type>



Получение информации по объектам.

    kubectl describle <type> <name>

Получить информацию об различных объектах.

    kubectl api-resources

Откат Deployment.

    kubectl rollout undo deployment <name>

Просмотр небольшой документации по различным объектам.

    kubectl explain <type>



