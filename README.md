#  Kittygram
## Запуск в контейнерах Docker и организация CI/CD с помощью GitHub Actions

### 1. Создание и развертывание контейнеров проекта:

Форкнуть репозиторий, склонировать и перейти в корень папки `kittygram_final`

Переименовать .env.example в .env и заполнить своими данными по шаблону:

```
mv .env.example .env
nano .env
```

Убедиться, что сервис (демон) Docker активен в системе:

```
sudo systemctl status docker
```

Развернуть контейнерное пространство:

(!) если развертка происходит впервые:

- необходимо открыть docker-compose.yml и заменить для каждого из контейнеров `backend`,  `frontend`, `gateway` способ выбора образа с `image` на `build` - таким образом будут созданы исходные docker образы проекта
- далее возможно создать свои образы `backend`,  `frontend`, `gateway` и использовать параметр `image`

```
nano docker-compose.yml
```
```
....
backend:
   ...
   # эту строку ниже:
   image: <username>/kittygram_backend
   # заменить на:
   build: ./backend/
   ...
frontend:
   ...
   # эту строку ниже:
   image: <username>/kittygram_frontend
   # заменить на:
   build: ./frontend/
   ...
gateway:
   ...
   # эту строку ниже:
   image: <username>/kittygram_gateway
   # заменить на:
   build: ./nginx/
   ...
```

Развернуть контейнерную группу в виде фоновоного демона:

```
docker compose up -d
```

Настроить миграции backend:

```
docker compose exec backend python manage.py migrate
```

Передать статику в Nginx:

```
docker compose exec backend python manage.py collectstatic
docker compose exec backend cp -r /kittygram/collect_static/. /backend_static/static/
```

Настроить Ваш сервер на отпраку запросов к сайту Kittygram на порт 9000 (согласно настройке образа `gateway`).
   
### 2. Создание CI/CD на GitHub Actions

Создать в корне проекта скрытую директиву для размещения файла с инструкциями для GitHub Actions:

```
mkdir .github/workflows
```

Переместить файл `kittygram_workflow.yml` в созданную директорию:

```
mv kittygram_workflow.yml ./.github/workflows/kittygram_workflow.yml
```

Перейти в раздел с проектом на GutHub -> Settings -> Secrets and variables -> actions

В разделе `secrets` создать (кнопка `New repository secret`) следующие "секреты":
- DOCKERHUB_USERNAME - имя пользователя DockerHub аккаунта
- DOCKERHUB_PASSWORD - пароль от DockerHub аккаунта
- HOST - IP вашего сервера
- SSH_KEY - SSH-ключ доступа к серверу, начиная с `-----BEGIN OPENSSH PRIVATE KEY-----` и заканчивая `-----END OPENSSH PRIVATE KEY-----`
- USER - имя пользователя учетной записи на сервере
- SSH_PASSPHRASE - пароль учетной записи на сервере
- TELEGRAM_TOKEN - токен TelegramBot, который будет уведомлять об успешном деплое проекта на сервере
- TELEGRAM_TO - ID пользователя Telegram, которому Bot будет присылать уведомлетие

### 3. Результаты работы

 При каждом push в  ветку main:
- запустятся автотесты приложений
- автоматически обновятся контейнеры
- контейнеры отправятся на DockerHub
- контейнеры скачаются с DockerHub на сервер и перезапустятся
- отправится уведомление пользователю через TelegramBot об успешном деплое на сервер

___

### Автор

Филипп Истомин