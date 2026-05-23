# Deployment Service

## Описание

Этот репозиторий содержит сервис управления жизненным циклом LLM-развертываний. Сервис хранит описание deployment-объектов, создает LLMDeployment в Kubernetes и координирует запуск проверки модели через validation service.

## Основные возможности
- создание и просмотр LLM-развертываний
- redeploy и удаление deployment-объектов
- сохранение metadata в PostgreSQL
- создание LLMDeployment CRD через Kubernetes client manager
- запуск validation run после поднятия модели
- служебные health/livez/service-info ручки

## Структура проекта

- `app/` — основной код приложения
  - `main.py` — FastAPI-приложение и HTTP-ручки
  - `config.py` — настройки сервиса

- `deploy/` — файлы и переменные для развертывания
- `.env.example` — пример переменных окружения
- `Dockerfile` — сборка Docker-образа
- `pyproject.toml` — зависимости и настройки Python-проекта
- `requirements.txt` — список зависимостей для совместимого запуска без uv

## Быстрый старт локально

1. Установите зависимости:
   ```bash
   uv sync
   ```

2. Создайте `.env` на основе `.env.example`:
   ```bash
   cp .env.example .env
   ```

3. Запустите сервис:
   ```bash
   uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
   ```

Если `uv` не используется, можно запустить через обычный virtualenv:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

## Переменные окружения
- `DATABASE_URL`
- `K8S_CLIENT_MANAGER_URL`
- `VALIDATION_SERVICE_URL`
- `SECURITY_SERVICE_URL`
- `SERVICE_TOKEN`
- `LOG_LEVEL`

Пример `.env`:

```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/llm_platform
SERVICE_TOKEN=change-me
LOG_LEVEL=INFO
```

## Основные API-ручки

| Метод | Ручка | Назначение |
|--------|-------|------------|
| `GET` | `/health` | Проверяет, что сервис доступен и может отвечать на management-запросы. |
| `GET` | `/livez` | Liveness probe для перезапуска контейнера при зависании процесса. |
| `GET` | `/service-info` | Возвращает имя сервиса, версию и базовую информацию для страницы статуса. |
| `GET` | `/deployments` | Отдает список LLM-развертываний с моделью, кластером, статусом и параметрами запуска. |
| `POST` | `/deployments` | Создает deployment metadata и отправляет LLMDeployment CRD в Kubernetes client manager. |
| `GET` | `/deployments/{deployment_id}` | Возвращает детальное состояние конкретного развертывания. |
| `DELETE` | `/deployments/{deployment_id}` | Удаляет deployment из control plane и инициирует удаление связанного LLMDeployment. |
| `POST` | `/deployments/{deployment_id}/redeploy` | Перезапускает существующее развертывание без создания новой бизнес-сущности. |
| `POST` | `/deployments/{deployment_id}/validate` | Запускает проверку модели через validation service после поднятия runtime workload. |

## Сборка и запуск в Docker

```bash
docker build -t hse-llm-project-2026/deployment_service:local .
docker run --env-file .env -p 8000:8000 hse-llm-project-2026/deployment_service:local
```

## Деплой в Kubernetes

Файлы развертывания лежат в папке `deploy/`. Для сервисов, которые уже подключены к стенду, используются Helm values и deploy-скрипты из соответствующего репозитория или общего инфраструктурного пайплайна.

## Метрики и документация

- Swagger UI: `/docs`
- OpenAPI: `/openapi.json`
- Health check: `/health`
- Liveness check: `/livez`

## Автор

Igor Malysh
