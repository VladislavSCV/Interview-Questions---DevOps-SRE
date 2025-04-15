## 🧠 Что такое CI/CD?

**CI** (Continuous Integration) — это практика автоматического тестирования и интеграции кода, когда разработчики регулярно (например, каждый день) сливают свои изменения в основную ветку.

**CD** (Continuous Delivery/Continuous Deployment) — это автоматическая доставка приложений на стадии продакшн или staging с минимальными усилиями, как только код проходит тесты.

---

## 🛠️ Основные компоненты GitLab CI/CD

1. **Pipeline** — это процесс, состоящий из последовательности задач, таких как сборка, тестирование и развертывание.
2. **Jobs** — это единичные шаги в пайплайне, такие как сборка, тестирование и развертывание.
3. **Stages** — это логическая группировка jobs в пайплайне (например, сборка, тестирование, деплой).
4. **Runners** — это агенты, которые выполняют jobs. Они могут быть локальными или облачными.

---

## 🧩 Основы `.gitlab-ci.yml`

Это конфигурационный файл для GitLab CI/CD. В нем описываются этапы пайплайна, jobs и их зависимости.

### Пример базового `.gitlab-ci.yml`:
```yaml
stages:
  - build
  - test
  - deploy

# Job для сборки приложения
build:
  stage: build
  script:
    - echo "Building the app..."
    - make build

# Job для тестирования
test:
  stage: test
  script:
    - echo "Running tests..."
    - make test

# Job для деплоя
deploy:
  stage: deploy
  script:
    - echo "Deploying the app..."
    - make deploy
  only:
    - master
```

- **stages** — последовательность этапов.
- **job** — для каждой задачи указываются действия (например, в **script**).
- **only** — указывает, на каких ветках/тегах будет запускаться job.

---

## 🚀 Как работает GitLab CI/CD

1. **Commit в репозиторий** — это инициирует запуск пайплайна в GitLab.
2. **Runner** выполняет job'ы в пайплайне:
   - Скачивает исходный код.
   - Выполняет команды, указанные в `script` (например, сборка, тесты).
3. **Пайплайн проходит через стадии**: каждая стадия (build, test, deploy) может включать несколько job'ов.
4. **Результат**:
   - Если все job'ы проходят, пайплайн считается успешным.
   - Если хотя бы один job не прошел, пайплайн будет помечен как неудачный, и нужно будет исправить ошибки.

---

## 🧩 Пример с несколькими job'ами и stages

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - echo "Building..."
    - docker build -t myapp .
  
test:
  stage: test
  script:
    - echo "Running tests..."
    - docker run myapp pytest

deploy:
  stage: deploy
  script:
    - echo "Deploying..."
    - kubectl apply -f k8s/deployment.yaml
  only:
    - master
```

### Важные моменты:
- **Docker** может быть использован для сборки и тестирования.
- Используем **kubectl** для деплоя в Kubernetes.

---

## 🧑‍💻 Важные директивы и параметры

1. **before_script** — выполняется до всех jobs в пайплайне:
   ```yaml
   before_script:
     - echo "Starting pipeline..."
   ```

2. **after_script** — выполняется после всех jobs:
   ```yaml
   after_script:
     - echo "Pipeline finished."
   ```

3. **cache** — для кэширования данных между пайплайнами (например, зависимости):
   ```yaml
   cache:
     paths:
       - node_modules/
   ```

4. **artifacts** — сохраняет файлы после выполнения job для последующего использования:
   ```yaml
   artifacts:
     paths:
       - build/
   ```

5. **dependencies** — для указания зависимостей между job'ами:
   ```yaml
   test:
     stage: test
     script:
       - run_tests
     dependencies:
       - build
   ```

---

## 💥 Развертывание в разные окружения

Пример развертывания в **staging** и **production**:

```yaml
stages:
  - build
  - test
  - staging
  - production

staging:
  stage: staging
  script:
    - echo "Deploying to staging"
    - kubectl apply -f staging-deployment.yaml
  only:
    - develop

production:
  stage: production
  script:
    - echo "Deploying to production"
    - kubectl apply -f production-deployment.yaml
  only:
    - master
```

- **develop** — ветка для staging.
- **master** — ветка для production.

---

## 🔒 Безопасность в CI/CD

1. **Секреты**: Используй **GitLab Secrets** для хранения переменных, таких как API ключи, пароли, токены.
   - Переменные могут быть настроены через **UI** или в файле `.gitlab-ci.yml`.

2. **Генерация токенов**:
   - Использование `CI_JOB_TOKEN` для авторизации и доступа в репозитории или сервисы.

---

## 🏃‍♂️ GitLab Runners

GitLab CI/CD использует **GitLab Runners** для выполнения jobs. Есть два типа runners:
1. **Shared runners** — общий runner, доступный всем проектам в GitLab.
2. **Specific runners** — настроенные для конкретного проекта.

### Регистрация GitLab Runner:
```bash
gitlab-ci-multi-runner register
```

1. Указываешь адрес GitLab.
2. Указываешь токен для регистрации.
3. Выбираешь executor (например, shell, docker).

---

## 🧑‍💻 Советы для CI/CD в GitLab

1. **Соблюдай принципы “Infrastructure as Code”** — скрипты для развертывания и тестирования должны храниться в репозитории.
2. **Не блокируй ветки** — не запрещай деплой на ветку, если это не критично.
3. **Используй версии** — для всех зависимостей используй конкретные версии.
4. **Отложенный деплой** — развертывай на staging, а потом на production.