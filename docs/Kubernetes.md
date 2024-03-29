# Kubernetes

# Содержание

* [Начало работы](#Начало-работы)
    * [](#)
    * [](#)

* [Базовые объекты](#базовые-объекты)
    * [Начальная конфигурация](#начальная-конфигурация)
    * [ReplicaSet](#replicaset)
    * [Namespace](#namespace)
    * [Deployment](#deployment)
        * [Обновление новых изменений](#обновление-новых-изменений)
        * [Работоспособность приложения](#работоспособность-приложения)
        * [Ограничение ресурсов](#ограничение-ресурсов)
    * [Service](#service)
    * [ConfigMap](#configmap)
    * [Secret](#secret)
    * [DaemonSet](#daemonset)
    * [ReplicationController](#replicationcontroller)
    * [](#)
    * [](#)

* [Команды](#команды)
    * [Apply](#apply)
    * [Delete](#delete)
    * [Get](#get)
    * [Logs](#logs)
    * [Describle](#describle)
    * [Port-forward](#port-forward)
    * [](#)


# Начало работы

## Установка


# Словарь

[Наверх](#содержание)

* `Манифест (manifest)` - это файл, в основном в формате yaml,
который содержит описание объектов Kubernetes,
такие как Pod, Deployment, Service, Ingress и т.д.

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

`Pod` - это минимальная единица для управления в Kubernetes.
Представляет собой один или несколько контейнеров,
предназначенный для определённой задачи.

Пример манифеста для Pod.

    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod                      # имя pod'a
      namespace: namespace-web          # указание namespace
    spec:                               # описание как должен выглядеть pod
      containers:
        - name: nginx-server
          image: nginx:latest
          ports:
            - containerPort: 80 # открытие порта 80 в контейнере


## ReplicaSet

[Наверх](#содержание)

`ReplicaSet` - обеспечивает масштабирование и управление однородными репликами подов (Pods).

Пример манифеста для ReplicaSet.

    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: nginx-rs
    spec:
      replicas: 3   # количество реплик Pod'ов, которые должны быть запущены
      selector:
        matchLabels:
          app: nginx    # селектор для выбора Pod'ов, которыми должен управлять ReplicaSet
      template:
        metadata:
          labels:
            app: nginx      # метки, которые будут присвоены Pod'ам
        spec:
          containers:
            - name: nginx-server
              image: nginx:latest
              ports:
                - containerPort: 80     # открытие порта 80 в контейнере


## Namespace

[Наверх](#содержание)

`Namespace (пространство имён)` - это виртуальная группа ресурсов,
которая обеспечивает изоляцию и разделение ресурсов между различными
проектами, командами или приложениями.

Пример манифеста для Namespace.

    apiVersion: v1
    kind: Namespace
    metadata:
      name: namespace-name


## Deployment

[Наверх](#содержание)

`Deployment (развертывание)` - позволяет управлять масштабированием и обновлением приложения.
Deployment обеспечивает создание и управление ReplicaSet'ами и Pod'ами.
Deployment гарантирует, что указанное количество реплик Pod'ов
всегда работает и доступно для обработки запросов в любое время.

Deployment позволяет выполнять масштабирование приложения,
изменять количество реплик Pod'ов,
выполнять обновление приложения с минимальным влиянием на работу сервиса
и откатывать изменения в случае необходимости.

Deployment использует ReplicaSet для создания и управления репликами Pod'ов,
а также для обеспечения надежности и отказоустойчивости.

Пример манифеста для Deployment.

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
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
            - name: nginx
              image: nginx:latest
              ports:
                - containerPort: 80


### Обновление новых изменений

[Наверх](#содержание)

`Strategy` - указывает стратегию, используемую для замены старых подов на новые.

1. `Recreate` - удаляет старые pods и создаёт новые. Может приводить к простою в приложении.

*Пример:*

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 3
      strategy:
        type: Recreate
      selector:
        matchLabels:
          app: nginx-server
      template:
        metadata:
          labels:
            app: nginx-server
        spec:
          containers:
            - name: nginx
              image: nginx:latest
              ports:
                - containerPort: 80

2. `RollingUpdate` - развёртывание обновляет pods в режиме непрерывного обновления.

У него есть 2 параметра:

* `maxUnavailable` - максимальное количество недоступных реплик во время обновления.
* `maxSurge` - максимальное количество дополнительных реплик,
которые могут быть созданы во время обновления.

*Пример:*

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 3
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxUnavailable: 1
          maxSurge: 1
      selector:
        matchLabels:
          app: nginx-server
      template:
        metadata:
          labels:
            app: nginx-server
        spec:
          containers:
            - name: nginx
              image: nginx:latest
              ports:
                - containerPort: 80


### Работоспособность приложения

[Наверх](#содержание)

Есть три типа проб, которые могут использоваться для проверки работоспособности приложения:

* `Liveness Probe` - используется для проверки того, работает ли приложение.
Если не работает, контроллер Kubernetes перезапустит Pod.
Это полезно в случае, если приложение периодически зависает или имеет другие проблемы,
которые могут привести к его аварийному завершению.
* `Readiness Probe` - используется для проверки того, готово ли приложение принимать трафик.
Если приложение не готово, Kubernetes не направит на него трафик.
Это может быть полезно в случае, если приложение требует некоторого времени для инициализации
или требует подключения к другим сервисам, прежде чем оно будет готово для обработки запросов.
* `Startup Probe` - используется для проверки того, что приложение успешно запустилось
и готово принимать запросы. Выполняется только один раз при запуске
и не проверяет состояние приложения на протяжении всего его жизненного цикла.

Также есть 3 способа проверки:

* `HTTP GET запрос` - проверка ответа на HTTP GET запрос к конечной точке приложения.

*Пример:*

    livenessProbe:
      httpGet:
        path: /healthcheck
        port: 80

* `TCP сокет` - роверка наличия открытого TCP сокета на определенном порту.

*Пример:*

    livenessProbe:
      tcpSocket:
        port: 8080

* `Exec команда` - проверка определённой команды внутри конетейнера.

*Пример:*

    livenessProbe:
      exec:
        command:
        - sh
        - -c
        - ps aux | grep nginx

Настройки Probes:

* `initialDelaySeconds` - определяет, сколько времени следует ждать после запуска контейнера,
прежде чем начинать проверку. Это позволяет приложению запуститься и готовиться к работе.
Значение этого параметра задается в секундах.
* `periodSeconds` определяет, как часто должна выполняться проверка после ее первого запуска.
Значение этого параметра также задается в секундах.
* `timeoutSeconds` - определяет время ожидания ответа на выполнение.
Если ответ не будет получен в течение этого времени, проверка считается неуспешной.
* `successThreshold` - определяет количество успешных проверок,
после которых контроллер Kubernetes считает приложение работающим.
* `failureThreshold` - определяет количество неудачных попыток выполнения,
после которых контроллер Kubernetes перезапустит контейнер.
* `terminationGracePeriodSeconds` - определяет время, которое будет предоставлено контейнеру
на завершение работы после получения сигнала завершения.
Если контейнер не завершит работу в течение этого времени,
он будет остановлен с помощью сигнала SIGKILL.

*Пример:*

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
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
            - name: nginx
              image: nginx:latest
              ports:
                - containerPort: 80
              startupProbe:
                httpGet:
                  path: /healthcheck
                  port: 80
                failureThreshold: 30
                periodSeconds: 10
              livenessProbe:
                httpGet:
                  path: /healthcheck
                  port: 80
                initialDelaySeconds: 30
                periodSeconds: 10
              readinessProbe:
                httpGet:
                  path: /healthcheck
                  port: 80
                initialDelaySeconds: 30
                periodSeconds: 10


### Ограничение ресурсов

[Наверх](#содержание)

Ограничение ресурсов нужно для того чтобы определить максимальное количество ресурсов,
которые могут быть использованы контейнером приложения,
и предотвращает перегрузку узла кластера.

* `Requests` - определяет минимальное количество ресурсов,
которое должно быть выделено для контейнера, чтобы он мог работать без задержек.
Если узел не имеет достаточно ресурсов, под не будет запущен на этом узле.
* `Limits` - определяет максимальное количество ресурсов,
которое может быть использовано контейнером.
Контейнер не может использовать более заданного количества ресурсов,
даже если они доступны на узле.

*Пример:*

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
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
            - name: nginx
              image: nginx:latest
              ports:
                - containerPort: 80
              resources:
                requests:
                  memory: "64Mi"
                  cpu: "250m"
                limits:
                  memory: "128Mi"
                  cpu: "500m"


## Service

[Наверх](#содержание)

`Service` - объект, который определяет логический набор подов и политику доступа к ним.

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


* `ClusterIP` - тип Service по умолчанию.
Доступен только внутри кластера Kubernetes и имеет виртуальный IP-адрес,
который используется для доступа к подам, связанным с этим Service.
* `NodePort` - будет открыт на всех узлах кластера внешним образом,
и любой запрос на этот порт будет направлен на соответствующий порт в Service.
Это позволяет обращаться к Service извне кластера через IP-адрес узла и порт Service.
* `LoadBalancer` - будет создан балансировщик нагрузки,
который будет распределять запросы между подами, связанными с Service.
* `ExternalName` - позволяет создавать внешний DNS-запись,
которая будет ссылаться на внешний ресурс за пределами кластера (например, базу данных или API-сервис).


## ConfigMap

[Наверх](#содержание)

`ConfigMap` - используется для хранения конфигурационных данных,
параметры конфигурации, настройки и другие данные, которые не являются конфиденциальными.

*Пример:*

    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: nginx-config
    data:
      port: 80
      ip_address: 127.0.0.1
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
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
            - name: nginx
              image: nginx:latest
              ports:
                - containerPort: 80
              envFrom:
                - configMapRef:
                  name: nginx-config


## Secret

[Наверх](#содержание)

`Secret` - секреты предназначены для хранения конфиденциальных данных.

Типы секретов:

* `Opaque` - Произвольные данные для хранения.
* `service-account-token` - Токен для ServiceAccount авторизации.
* `dockercfg` - сериализованный `~/.dockercfg`.
* `dockerconfigjson` - сериализованный `~/.docker/config.json`.
* `basic-auth` - учетные данные для базовой аутентификации.
* `ssh-auth` - учетные данные для аутентификации SSH.
* `tls` - данные для клиента или сервера TLS.

Просмотр секрета.

    kubectl get secret <name> -o yaml


## DaemonSet

[Наверх](#содержание)


# Команды

## Apply

[Наверх](#содержание)



## Delete

[Наверх](#содержание)



## Get

[Наверх](#содержание)

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

## Logs

[Наверх](#содержание)

    kubectl logs <pod>

## Describle

[Наверх](#содержание)

Получение информации по объектам.

    kubectl describle <type> <name>

## Port-forward

[Наверх](#содержание)

kubectl port-forward service 8080:8080


## ofher

[Наверх](#содержание)

Получить информацию об различных объектах.

    kubectl api-resources

Откат Deployment.

    kubectl rollout undo deployment <name>

Просмотр небольшой документации по различным объектам.

    kubectl explain <type>
