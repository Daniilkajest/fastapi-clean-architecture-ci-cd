# Тестовое задание: DevOps (FastAPI + PostgreSQL in Kubernetes)

## 1. Инструкция по работе с CI/CD (Для разработчика)
Пайплайн настроен через GitHub Actions (файл `.github/workflows/deploy.yml`).
* **При создании Pull Request в ветку `main`:** Запускается стадия `test` (линтеры, юнит-тесты). Деплой не происходит.
* **При слиянии (Merge) в `main`:** Запускается полная сборка. Образ собирается локально, тегируется, загружается в кэш Minikube, после чего обновляются манифесты Kubernetes (БД и само приложение) в окружении `Development`.
* Для доступа к приложению после деплоя необходимо выполнить проброс порта: `kubectl port-forward service/fastapi-service 8000:80`. Swagger будет доступен по адресу `http://localhost:8000/docs`.

## 2. Архитектурные решения и их обоснование
* **GitHub Actions вместо GitLab CI:** Использован в качестве альтернативы из-за временных ограничений на бесплатную регистрацию в SaaS-версии GitLab (gitlab.com) без иностранной карты/номера. Логика пайплайна (test -> build -> deploy) полностью сохранена и тривиально переносится в синтаксис `.gitlab-ci.yml`.
* **Локальный Self-hosted Runner и Minikube:** Так как облачный кластер не предоставлялся, вся инфраструктура развернута локально. Runner исполняет команды напрямую в хост-системе (Windows + PowerShell), имея доступ к `docker` и `minikube`.
* **Доставка образов без Registry:** Из-за использования Minikube с драйвером Docker Desktop, образы собираются системным Docker и загружаются напрямую в кластер через `minikube image load`. Это избавляет от необходимости поднимать локальный Registry или бороться с rate-limits Docker Hub.

## 3. Что бы я сделал иначе при наличии ресурсов
1. **Helm-чарты:** Заменил бы сырые YAML-манифесты на Helm. Это позволит легко шаблонизировать переменные (имена хостов, ресурсы, реплики) для будущих окружений (staging, production).
2. **Управление секретами:** Отказался бы от ручного создания Kubernetes Secrets в пользу интеграции с HashiCorp Vault (через Vault Injector) или External Secrets Operator.
3. **Container Registry:** Использовал бы полноценный реестр (например, встроенный GitLab Container Registry) для хранения версионированных Docker-образов.
4. **Ingress Controller:** Вместо `NodePort` и ручного `port-forward` настроил бы Nginx Ingress, чтобы сервис был доступен снаружи кластера по доменному имени.

## 4. Вопросы к команде перед внедрением в реальный проект
1. Какую стратегию ветвления использует команда (GitFlow, Trunk-based) и как это должно маппиться на наши окружения (dev, stage, prod)?
2. Где физически развернут Production кластер и кто отвечает за управление Ingress/DNS?
3. Как организован бэкап PostgreSQL? Используются ли операторы (например, Zalando или CloudNativePG) или это managed-решение от облачного провайдера?
4. Есть ли у нас выделенное хранилище секретов (Vault) для хранения `DB_PASSWORD`?





# fastapi-clean-architecture

## description
Base FastAPI project for applying general RestAPI Application cases.
![openapi-docs](./doc/images/openapi-docs-v2.png)

## concept
1. Minimal functionality.
2. Convincing architecture.
3. Easy to read.
4. Compatibility.
5. Versatility.

## base models
1. user
2. post [user (1 : n) post]
3. tag [post (n : n) tag]

## integrated with
1. Python3.9+
2. Fastapi 0.78.0
3. Database
   1. MySQL5.7+
   2. Migration with alembic
   3. pytest with real DB
   4. Load with two ways (eager, lazy)
   5. Modeling with schema (1:1, 1:n, n:n)
4. dependency-injector
   1. service-repository pattern
5. JWT authentication
   1. role separation each endpoint
6. Deployment
   1. container environment(k8s, docker)
   2. raw WAS(Web Application Server)

## commands
1. db(alembic)
   1. `alembic upgrade head`: apply every migrations
   2. `alembic downgrade base`: rollback every migrations
   3. `alembic revision --autogenerate -m "revision_name"`: create new migration 
   4. `alembic history`: get alembic revision history
2. How to migration
   1. Create or modify models from `app/model/*.py`
   2. `alembic -x ENV=[dev|stage|prod] revision --autogenerate -m "revision_name"`
   3. Check auto generated migration file from `app/migrations/versions/*.py`
   4. `alembic -x ENV=[dev|stage|prod] upgrade head`  
      If ENV does not exist, it will be applied to the test.
3. server
   1. `uvicorn app.main:app --reload`: base
   2. options
      1. host: `--host 0.0.0.0`
      2. port: `--port 8000`
4. test
   1. `pytest`: base 
   2. `pytest --cov=app --cov-report=term-missing`: coverage with stdout
   3. `pytest --cov=app --cov-report=html`: coverage with html

## sample env
```dotenv
# mysql case
ENV=dev
DB=mysql
DB_USER=root
DB_PASSWORD=qwer1234
DB_HOST=localhost
DB_PORT=3306

# postgres case
ENV=dev
DB=postgresql
DB_USER=gyu
DB_PASSWORD=
DB_HOST=localhost
DB_PORT=5432
```

## references
1. [FastAPI official docs](https://fastapi.tiangolo.com/)
2. [alembic official tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html)
3. [Dependency Injector](https://python-dependency-injector.ets-labs.org/)
