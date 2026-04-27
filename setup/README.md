# Setup - запуск окружения

Здесь минимальный docker-compose для self-hosted работы над заданием. Поднимает postgres, n8n, minio. Никакого Traefik, никаких SSL, никакой SSH-инфры. Только то, что нужно, чтобы импортировать воркфлоу и потыкать.

## Требования

- Docker и docker-compose установлены.
- Свободные порты: 5432 (postgres), 5678 (n8n), 9000 / 9001 (minio).

## Запуск

```bash
cd setup
docker compose up -d
```



Проверка:
```bash
docker compose ps              # все дерево контейнера healthy
docker compose logs n8n        # последние строки - n8n ready, listening on 5678
```

## Первый вход в n8n

1. Открой `self-hosted n8n`.
2. Создай owner-аккаунт, если нет.
3. Слева в меню: Credentials.

## Создание credentials в n8n

В воркфлоу из задания используются три credential. Нужно создать их в UI:

### Postgres "test_pg"

- Type: Postgres
- Host: `postgres`
- Database: `tz_test`
- User: `postgres`
- Password: `testpass`
- Port: `5432`
- SSL: Disable

Имя credential: `test_pg`. Если назвать иначе - потом в импортированных воркфлоу нужно будет вручную перевесить credential на нодах Postgres.

### S3 "test_s3"

- Type: S3
- Region: `us-east-1` (любой)
- Access Key Id: `minioadmin`
- Secret Access Key: `minioadmin`
- Endpoint: `http://minio:9000`
- Force Path Style: Enable

Имя credential: `test_s3`.

### HTTP Header Auth "test_sendgrid" (для 01-mini-register)

- Type: Header Auth
- Name: `Authorization`
- Value: `Bearer SG.fake_dev_only_replaceme`

Имя: `test_sendgrid`. Реально дёргать SendGrid не нужно, смотри README заданий, это для красоты импорта.

## Импорт воркфлоу

В n8n UI: **Workflows → Import from file** (или drag-and-drop). По одному JSON из `../part-1-code-review/workflows/` и `../part-2-forensic/`.

После импорта откроешь каждый, проверишь, что credentials на нодах подвязаны к твоим. Если нет - перевешиваешь руками.

## MinIO consoлe

Открой `http://your-domen:9001` или `http://localhost:9001` (если локально), логин/пароль из `.env`. Создай бакет `uploads` для воркфлоу 03-file-uploader.

## Остановка и сброс

```bash
docker compose down              # остановить, данные сохранить
docker compose down -v           # остановить и удалить volumes, полностью сбросит
```

## Что-то не работает

- n8n не стартует - проверь, что postgres healthy: `docker compose ps`. Если postgres unhealthy - смотри логи `docker compose logs postgres`.
- Не могу подключиться к Postgres из n8n - host должен быть `postgres`, а не `localhost`, это внутренняя сеть docker.
- MinIO - host тоже `minio`, не `localhost`.
- Воркфлоу импортировался, но при запуске ругается на credentials - перевесь вручную в каждой ноде.
