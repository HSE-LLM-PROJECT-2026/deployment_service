# Deployment Service

## Описание

Сервис управления жизненным циклом деплойментов моделей: создание, просмотр, удаление, redeploy и запуск валидации.

## Основные возможности

- создание и удаление deployment-записей
- запуск redeploy и validation
- публикация service-info/health для control plane

## Структура проекта

- `app/` - код сервиса (FastAPI, config, domain handlers)
- `deploy/` - служебные файлы для роли сервиса в деплое
- `pyproject.toml` - зависимости и метаданные проекта
- `Dockerfile` - сборка контейнера
- `.env.example` - пример переменных окружения

## Быстрый старт (локально)

1. Установить зависимости:
   `uv sync --frozen --extra dev`
2. Запустить сервис:
   `uv run uvicorn app.main:app --host 0.0.0.0 --port 8000`
3. Проверить health:
   `curl http://127.0.0.1:8000/health`

## Переменные окружения

- `SERVICE_ROLE` - роль сервиса в control plane
- `SERVICE_NAME` - техническое имя сервиса
- `POSTGRES_DSN` - строка подключения к PostgreSQL
- `PROMETHEUS_BASE_URL` - адрес Prometheus
- `SERVICE_TO_SERVICE_URLS_JSON` - карта внутренних URL сервисов

## Docker

- Сборка: `docker build -t deployment_service:local .`
- Запуск: `docker run --rm -p 8000:8000 --env-file .env deployment_service:local`

## Деплой

Файлы для деплоя лежат в `deploy/`.

## Основные API ручки

- `GET /deployments`
- `POST /deployments`
- `GET /deployments/{deployment_id}`
- `DELETE /deployments/{deployment_id}`
- `POST /deployments/{deployment_id}/redeploy`
- `POST /deployments/{deployment_id}/validate`
