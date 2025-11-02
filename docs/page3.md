# Helm

Helm — это **менеджер пакетов (package manager)** для Kubernetes, похожий на `apt`/`yum` в Linux или `npm` в Node.js. Он позволяет:

- Упаковывать приложения в **чарты (charts)** — готовые шаблоны развертывания.
- Управлять зависимостями между компонентами приложения.
- Легко настраивать приложения через **values.yaml**.
- Развертывать, обновлять и откатывать приложения одной командой.

## Основные понятия Helm

- **Chart** — пакет с описанием Kubernetes-ресурсов (Deployment, Service, ConfigMap и т. д.).
- **Release** — экземпляр чарта, запущенный в кластере.
- **Repository (repo)** — хранилище чартов (как репозиторий Docker, но для Helm).
- **Values** — переменные для кастомизации чарта.

## Зачем нужен Helm?

**Без Helm** развертывание приложения в Kubernetes выглядит так:

```bash
kubectl apply -f deployment.yaml  
kubectl apply -f service.yaml  
kubectl apply -f configmap.yaml  
```
... и так для каждого файла  

**Проблемы:**

- Трудно управлять множеством YAML-файлов.
- Нет простого способа параметризировать конфигурации (например, для разных окружений).
- Сложно обновлять и откатывать приложения.
    

**С Helm:**

```bash
helm install my-app ./my-chart  
```

- Все ресурсы упакованы в один чарт.
- Конфигурация задается через `values.yaml`.
- Можно обновлять (`helm upgrade`) и откатывать (`helm rollback`) приложения.

**Когда используется:**

- Если у вас **много YAML-файлов** и нужно их упростить.
- Если вы **деплоите одно приложение в разные окружения** (dev/stage/prod).
- Если хотите **использовать готовые чарты** (например, для Prometheus, Grafana, PostgreSQL).

## С чего начать изучение Helm?

Установка на Ubuntu:

```console
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

### Основные команды

| Команда                                                    | Описание                      |
| ---------------------------------------------------------- | ----------------------------- |
| `helm version`                                             | Проверить версию Helm         |
| `helm repo add bitnami https://charts.bitnami.com/bitnami` | Добавить репозиторий          |
| `helm search repo bitnami/nginx`                           | Найти чарт                    |
| `helm install my-nginx bitnami/nginx`                      | Установить чарт               |
| `helm list`                                                | Показать установленные релизы |
| `helm upgrade my-nginx bitnami/nginx`                      | Обновить релиз                |
| `helm uninstall my-nginx`                                  | Удалить релиз                 |
| `helm create my-chart`                                     | Создать свой чарт             |

### Создание своего чарта

```
helm create my-chart          # Создает структуру чарта
```

Структура чарта:

```
my-chart/  
├── charts/              # Зависимости  
├── Chart.yaml           # Метаданные чарта  
├── templates/           # Шаблоны Kubernetes-ресурсов  
│   ├── deployment.yaml  
│   ├── service.yaml  
│   └── ...  
└── values.yaml          # Конфигурация по умолчанию  
```

Файл Chart.yaml с большой буквы. Его содержание:

```yaml
apiVersion  : v2
name        : App-HelmChart
description : my first chart
type        : application
version     : 0.1.0
appVersion  : 1.0.0
```

Файл `values.yaml` содержит необходимые дефолтные переменные - количество реплик контейнера и сам образ:

```yaml
container:
	image: dockerhub/blablabla:latest

replicaCount: 2
```

Теперь деплоим это не через kubectl, а через helm:
`helm install <название> <папка_с_чартом>/`
пример:
`helm install App App-HelmChart/`

Можно создать собственный файл с дополнительными переменными, например `prod_values.yaml` , и указать его при запуске: 
`helm install App App-HelmChart/ -f prod_values.yaml`

!!! note

    note note note