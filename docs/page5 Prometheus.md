# Prometheus
# 1. Теоретическая основа Prometheus

**Prometheus** — это open-source система мониторинга и оповещения.

Простыми словами: **Prometheus — это "сторож" для ваших приложений и инфраструктуры.** Он постоянно спрашивает "как дела?" у всех ваших сервисов, собирает их ответы (метрики), складывает в свою базу и позволяет вам анализировать эти данные, строить графики и отправлять уведомления, если что-то пошло не так.

## 1.1 Основные задачи

- **Сбор метрик:** Получение числовых данных о работе системы (например, загрузка CPU, потребление памяти, количество запросов к веб-серверу, время ответа базы данных).
    
- **Хранение метрик:** Все собранные данные хранятся в специальной высокопроизводительной временной базе данных (Time Series DB).
    
- **Запросы и анализ:** С помощью мощного языка запросов **PromQL** можно извлекать и агрегировать данные, чтобы находить закономерности и проблемы.
    
- **Визуализация:** Построение графиков и дашбордов на основе собранных данных. Чаще всего для этого используют **Grafana**, которая идеально дружит с Prometheus.
    
- **Оповещение (Alerting):** Настройка правил, которые при превышении порогового значения (например, "ошибок больше 5%") отправляют уведомления в Slack, Email, Telegram и т.д.

## 1.2. Как работает Prometheus? Ключевые принципы

Новичку важно понять две ключевые концепции:

#### **a) Pull-модель (Вытягивание метрик)**

Это **главное отличие** Prometheus от многих других систем (например, Zabbix).

- **Как у других (Push):** Приложения сами отправляют (push) метрики на сервер мониторинга.
    
- **Как у Prometheus (Pull):** Сам **Prometheus-сервер** периодически обращается (вытягивает, scrape) к списку своих целей (targets) по HTTP и забирает метрики.
    

**Преимущества Pull-модели:**

- Можно легко запустить сервер мониторинга на своем ноутбуке и "дергать" сервисы для отладки.
    
- Не нужно заботиться о том, куда приложению отправлять метрики; оно просто предоставляет endpoint.
    
- Легко понять, "мертв" ли target (Prometheus не может до него достучаться).
    

#### **b) Многомерная модель данных**

Метрики в Prometheus — это не просто `cpu_usage = 45%`. Это наборы пар ключ-значение (labels), что делает данные невероятно гибкими.

Пример метрики:

text

`http_requests_total{method="POST", handler="/api/users", status="200", instance="10.0.0.1:8080"} 1500`

Здесь:

- `http_requests_total` — имя метрики.
- `method`, `handler`, `status`, `instance` — **лейблы** (labels). Они описывают измерение.
- `1500` — значение.

Благодаря лейблам можно делать запросы типа:

- "Покажи все запросы к `/api/users`"
- "Покажи все ошибки 500 на всех инстансах"
- "Сравни количество GET и POST запросов"

Это невероятно мощный инструмент для анализа.

### 1.3. Ключевые компоненты, которые нужно знать

Вся экосистема Prometheus состоит из нескольких частей:

1. **Prometheus Server:** Само ядро. Занимается сбором (scraping) и хранением метрик.
	В свою очередь Сервер включает в себя:
	
	- **Хранилище**, которое собирает метрики
		
	- **HTTP Server**, поддерживающий PromQL запросы, web-UI, отображает графану.
		
	- **Retrieval** - сервис по сбору метрик, который ходит по конечным точкам и собирает метрики, помещая их затем в хранилище

2. **Client Libraries:** Библиотеки для разных языков программирования (Go, Java, Python, Ruby и др.), которые встраиваются в код вашего приложения для сбора и предоставления метрик. Ваше приложение должно предоставлять endpoint (обычно `/metrics`), откуда Prometheus будет забирать данные.
    
3. **Exporters:** Отдельные приложения, которые нужны, чтобы мониторить системы, которые сами не отдают метрики в формате Prometheus. Например:
    
    - **Node Exporter:** для мониторинга аппаратного обеспечения и ОС (Linux, Windows).
        
    - **cAdvisor / Docker Exporter:** для мониторинга контейнеров.
        
    - **Blackbox Exporter:** для мониторинга доступности сайтов, портов (HTTP, TCP, ICMP).
        
4. **Pushgateway:** Особый компонент для кратковременных задач (cron jobs), которые не живут долго и не могут ждать, пока их "выдернут". Они отправляют (push) свои метрики в Pushgateway, а Prometheus уже забирает их оттуда.
    
5. **Alertmanager:** Отдельный сервис, который обрабатывает оповещения от Prometheus Server, группирует их, убирает дубли и отправляет по нужным каналам (Slack, email и т.д.).
    
6. **Grafana:** Не является частью Prometheus, но это лучший друг. Используется для создания красивых и информативных дашбордов на основе данных из Prometheus.

# 2. Практика (чистый prometheus)

https://www.youtube.com/playlist?list=PLg5SS_4L6LYu6qjwwwjt2WRsEudkzqB7J

## 2.1. Подготовка и активация

1) Устанавливаем Priometheus на ubuntu:
	`wget <ссылка на архив с нужной версией .tar.gz>`

2) Распаковываем архив:
	 `tar xvfz archive.tar.gz`

3) В распакованном архиве можно удалить файлы LICENSE, NOTICE. 
	 `rm LICENSE`
	 `rm NOTICE `

4) Откроем и отредактируем через nano файл `prometheus.yml` . Можно подчистить все комментарии-подсказки и оставить необходимое:

```yaml
# интервал сбора данных
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "myprometheus"

# ui с данными. сюда же добавляется рабочий ip для отслеживания.
    static_configs:
      - targets: ["localhost:9090"]
	        # - localhost:9090
	        # - 10.10.0.1.:9090
```

5) Активируем Prometheus:
	 `./prometheus`

Видим много букв. На порте 9090 видим web-ui с данными, пока не прервём процесс ctrl + C.
Параллельно с этим в нашей рабочей директории создается папка с БД `/data`. Все метрики сохраняются там.

## 2.2. Переделываем по-взрослому: автозапуск

В реальности необходимо держать prometheus в рабочем состоянии постоянно или хотя бы запускать его одновременно с запуском машины. 
Хранение данных тоже должно находиться в специализированном месте.

```bash
sudo su

# Переносим бинарник prometheus*. После этого prometheus можно будет запускать из любой директории:
mv prometheus /usr/bin

# Создаем директорию для данных
mkdir /etc/prometheus/data

# Переносим prometheus.yml в созданную директорию (или в другую по надобности):
mv /etc/prometheus

# Можно удалить папку /prometheus из корневой директории
rm -rf /prometheus

# Добавляем системного юзера prometheus и передаем в его владение бинарник prometheus:
useradd -rs /bin/false prometheus
chown prometheus:prometheus /usr/bin/prometheus
chown -R prometheus:prometheus /etc/prometheus
```

А теперь самое главное: создаем из бинарника СЕРВИС, чтобы он запускался при запуске машины. Для этого создаем файл:
`nano etc/systemd/system/prometheus.service`

Его содержимое:

```
[Unit]
Description=Prometheus Server
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
ExecStart=/usr/bin/prometheus \
	--config.file        /etc/prometheus/prometheus.yml \
	--storage.tsdb.path  /etc/prometheus/data

[Install]
WantedBy=multi-user.target
```

```bash
# Теперь делаем reload и стартуем сервис:
systemctl daemon-reload
systemctl start

# Проверяем работает ли наш сервис:
systemctl status prometheus

# Делаем запуск сервиса автоматическим при запуске машины:
systemctl enable prometheus
```

Полный готовый скрипт можно скачать из гитхаба adv-it.

# 3. Установка Prometheus Operator в облачный кластер через HELM.

По традиции для разворачивания мониторинга через прометеус можно собрать вместе все *.yaml* декларации для каждого компонента и разворачивать их в нужной последовательности, что неудобно.

Вместо этого можно использовать **Operator**, который в сути своей является менеджером компонентов прометеуса. Он управляет порядком запуска. Достаточно будет найти подходящий оператор и развернуть его в кластере.

Но наиболее эффективным методом является использование **HELM-чарта**, который в свою очередь введёт в действие подходящий оператор. То есть, HELM формирует сетап, а оператор им управляет. Мы будем использовать версию comunity.

Репозиторий:
https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack#kube-prometheus-stack

// Устанавливаем чарт Prometheus-comunity в кластер:

```bash
tiv@DESKTOP-0A8EL6K:~$ helm install moe-release oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack

# (moe-release - имя релиза, которое хотим)
# Вывод команды:

Pulled: ghcr.io/prometheus-community/charts/kube-prometheus-stack:77.9.1
Digest: sha256:508745ffc3470483396e1639189443325cfb291ade3fb4987c08accbdabc0710
NAME: moe-release
LAST DEPLOYED: Fri Sep 19 17:33:41 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace default get pods -l "release=moe-release"

Get Grafana 'admin' user password by running:

  kubectl --namespace default get secrets moe-release-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

Access Grafana local instance:

  export POD_NAME=$(kubectl --namespace default get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=moe-release" -oname)
  kubectl --namespace default port-forward $POD_NAME 3000

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

Здесь разработчики уже предлагают проверить командой успешную установку. Видим, что логин от графаны: **admin**. Пароль можно получить командами ниже: `kubectl...kubectl...export...kubectl...` 
В общем делаем всё по инструкции из терминала. Port-forward переводит *Grafana* на *localhost:3000*.
Выполняем:

```bash
tiv@DESKTOP-0A8EL6K:~$ kubectl --namespace default get pods -l "release=moe-release"

NAME                                                   READY   STATUS    RESTARTS   AGE
moe-release-kube-prometheu-operator-57b44c7bcc-xk7dc   1/1     Running   0          26s
moe-release-kube-state-metrics-669f47f9bd-hz2p7        1/1     Running   0          26s
moe-release-prometheus-node-exporter-2fvb5             1/1     Running   0          26s
moe-release-prometheus-node-exporter-mcd78             1/1     Running   0          26s

tiv@DESKTOP-0A8EL6K:~$ kubectl --namespace default get secrets moe-release-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo
prom-operator

tiv@DESKTOP-0A8EL6K:~$ export POD_NAME=$(kubectl --namespace default get pod -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=moe-release" -oname)

tiv@DESKTOP-0A8EL6K:~$ kubectl --namespace default port-forward $POD_NAME 3000
Forwarding from 127.0.0.1:3000 -> 3000
Forwarding from [::1]:3000 -> 3000
Handling connection for 3000

```

До прерывания ctrl + C на 3000 порте localhost будет доступен UI графаны. 
**Великолепно!**

---

## Упражнение: port-forwarding

Переведём и другие сервисы на какие-либо порты.

**Общий синтаксис команды**: 

`kubectl port-forward [resource-type]/[resource-name] [local-port]:[resource-port]`

**Объяснение**:

- resource-type — тип ресурса (например, pod, svc).
- resource-name — имя ресурса.
- local-port — локальный порт на компьютере.
- resource-port — порт на ресурсе Kubernetes.
Можно одновременно через пробел добавить дополнительную пару портов.


```bash
# смотрим названия сервисов и их порты
tiv@DESKTOP-0A8EL6K:~$ kubectl get services

NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
alertmanager-operated                     ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   19m
kubernetes                                ClusterIP   10.96.128.1     <none>        443/TCP                      40m
moe-release-grafana                       ClusterIP   10.96.157.175   <none>        80/TCP                       19m
moe-release-kube-prometheu-alertmanager   ClusterIP   10.96.214.51    <none>        9093/TCP,8080/TCP            19m
moe-release-kube-prometheu-operator       ClusterIP   10.96.206.68    <none>        443/TCP                      19m
moe-release-kube-prometheu-prometheus     ClusterIP   10.96.151.16    <none>        9090/TCP,8080/TCP            19m
moe-release-kube-state-metrics            ClusterIP   10.96.137.236   <none>        8080/TCP                     19m
moe-release-prometheus-node-exporter      ClusterIP   10.96.137.177   <none>        9100/TCP                     19m
prometheus-operated                       ClusterIP   None            <none>        9090/TCP                     19m

# prometheus
tiv@DESKTOP-0A8EL6K:~$ kubectl port-forward svc/moe-release-kube-prometheu-prometheus 9999:9090
Forwarding from 127.0.0.1:9999 -> 9090
Forwarding from [::1]:9999 -> 9090
Handling connection for 9999

# alertmanager
tiv@DESKTOP-0A8EL6K:~$ kubectl port-forward svc/moe-release-kube-prometheu-alertmanager 9093:9093
Forwarding from 127.0.0.1:9093 -> 9093
Forwarding from [::1]:9093 -> 9093
Handling connection for 9093
```

---
Применить изменения к чарту в кластере: ???
`helm install update tivzrelease -f values.yaml`