#CI-CD

# 1. База

## 1.1. Что такое CI/CD?

**CI/CD** — это практика непрерывной интеграции и непрерывной доставки/развертывания.

- **CI (Continuous Integration)** — Непрерывная интеграция
    - Разработчики часто сливают свои изменения в общий репозиторий
    - Каждое изменение автоматически проверяется

- **CD (Continuous Delivery/Deployment)** — Непрерывная доставка/развертывание
    - **Continuous Delivery**: код всегда готов к выпуску в production
    - **Continuous Deployment**: код автоматически выпускается в production

## 1.2. Ключевые понятия, которые нужно знать

### Основные компоненты:

- **Pipeline** — автоматизированный процесс сборки, тестирования и развертывания
- **Build** — процесс компиляции кода в исполняемые артефакты
- **Test** — автоматизированное тестирование
- **Deploy** — развертывание приложения на серверах

### Типы тестов в CI/CD:

- **Unit tests** — тестирование отдельных компонентов
- **Integration tests** — тестирование взаимодействия компонентов
- **E2E tests** — сквозное тестирование всего приложения

## 1.3. Популярные инструменты

- **GitHub Actions** — интегрирован в GitHub
- **GitLab CI** — встроен в GitLab
- **Jenkins** — классический, гибкий, но требует настройки
- **CircleCI** — популярный облачный вариант

## 1.4. Базовый workflow CI/CD

Схема типичного пайплайна выглядит так:

```
1. Developer pushes code → Git repository
2. CI server detects changes
3. Build application
4. Run unit tests
5. Run integration tests
6. Build Docker image (опционально)
7. Deploy to staging
8. Run E2E tests
9. Deploy to production (автоматически или вручную)
```

**Пример простого pipeline (GitHub Actions):**

```yaml
name: CI Pipeline

on: [push]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
	
    - name: Install dependencies
      run: npm install
      
    - name: Run tests
      run: npm test
      
    - name: Build project
      run: npm run build
```

## 1.5. Best practice 

- ✅ Начинай с малого — сначала настрой тесты
- ✅ Используй кеширование для ускорения сборок
- ✅ Храни секреты в защищенных переменных
- ✅ Делай пайплайны быстрыми — не более 10 минут
- ✅ Добавляй проверки качества кода (линтеры)

 **Что изучить дальше:**
1. **Docker** — контейнеризация приложений
2. **Инфраструктура как код** (Terraform, Ansible)
3. **Мониторинг** пайплайнов
4. **Безопасность** в CI/CD (DevSecOps)

Ещё теория:
https://www.youtube.com/watch?v=pFKwmEdwZZQ&t=4s
# 2. GitLab CI

### Основные компоненты:

- **GitLab Runners** — серверы, которые выполняют jobs
- **.gitlab-ci.yml** — файл конфигурации пайплайна
- **Pipelines** — весь процесс сборки/тестирования/развертывания
- **Jobs** — отдельные задачи в пайплайне
- **Stages** — группы jobs, которые выполняются последовательно

### 📁 **Файл .gitlab-ci.yml**

Создаётся в корне репозитория:

```yaml
# Пример базового пайплайна
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"

# Job 1: Тестирование
unit_tests:
  stage: test
  image: node:$NODE_VERSION
  script:
    - npm install
    - npm run test:unit
  artifacts:
    paths:
      - coverage/
    expire_in: 1 week
  only:
    - merge_requests
    - main

# Job 2: Сборка
build_app:
  stage: build
  image: node:$NODE_VERSION
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  dependencies:
    - unit_tests
  only:
    - main

# Job 3: Деплой в staging
deploy_staging:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client
    - scp -r dist/ user@staging-server:/app/
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main
  when: manual  # Требует ручного запуска

# Job 4: Деплой в production
deploy_production:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to production..."
    - ./deploy-prod.sh
  environment:
    name: production
    url: https://example.com
  only:
    - tags  # Только при создании тега
  when: manual
```

## ⚙️**Ключевые директивы GitLab CI**

### Stages & Jobs:
```yaml
stages:
  - pre-test
  - test
  - build
  - deploy

job1:
  stage: pre-test
  script: echo "Running first"

job2:
  stage: test  
  script: echo "Running tests"
```
### Условия выполнения:
```yaml
# Только для конкретных веток
only:
  - main
  - develop

# Исключения
except:
  - tags

# Правила (более гибко)
rules:
  - if: $CI_COMMIT_BRANCH == "main"
  - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```
### Артефакты и кэши:
```yaml
build:
  script: npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
    
test:
  script: npm test
  cache:
    paths:
      - node_modules/
    key: ${CI_COMMIT_REF_SLUG}
```

## 🏃 **GitLab Runners**

### Типы раннеров:
- **Shared** — общие для всего GitLab инстанса
- **Group** — для группы проектов
- **Project** — специфичные для проекта
- **Specific** — для конкретных jobs
### Регистрация раннера:
```bash
# 1. Получить token из GitLab (Settings → CI/CD → Runners)
# 2. Запустить команду регистрации раннера, копируется из гитлаба

gitlab-runner register
  --url "https://gitlab.com"
  --registration-token "PROJECT_TOKEN"
  --executor "docker"
  --description "My Runner"
```
Эта команда превращает машину (вм или реальную) в раннер. Если раннер и проект находятся в одном окружении (Яндекс-клауд, гитлаб и тд), то специально их дружить не нужно. Если, например, кластер на Яндекс-клауде, а раннег на GKE, то политики подключения можно настроить кнопками в ide-шке.

## 🌟 **Продвинутые возможности**

### Включаемые шаблоны:
```yaml
include:
  - project: 'my-group/my-project'
    ref: main
    file: '/templates/.gitlab-ci-template.yml'
```
### Многопроектные пайплайны:

```yaml
deploy_app:
  stage: deploy
  trigger:
    project: my-group/deployment
    strategy: depend
```
### Параллельные jobs:

```yaml
parallel_tests:
  stage: test
  script: ./run-test.sh $CI_NODE_INDEX $CI_NODE_TOTAL
  parallel: 5  # Запуск 5 параллельных jobs
  ```
## 🛠️ **Переменные окружения**

### В GitLab UI: Settings → CI/CD → Variables

Пример:

```yaml
deploy_prod:
  script:
    - echo "Using secret: $API_KEY"
    - deploy --token $DEPLOY_TOKEN
  
  # Переменные задаются и защищаются в UI
```

## 📊 **Мониторинг и отладка**

### Просмотр пайплайнов:

- **Pipelines** → Список всех пайплайнов
- **CI/CD Charts** → Графики и метрики
- **Jobs** → Детали выполнения каждого job

 Полезные переменные для отладки:
```yaml
debug_info:
  script:
    - echo "Branch: $CI_COMMIT_REF_NAME"
    - echo "Commit: $CI_COMMIT_SHA"
    - echo "Pipeline: $CI_PIPELINE_ID"
```

