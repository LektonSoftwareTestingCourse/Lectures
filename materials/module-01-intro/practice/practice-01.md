# Practice 1: «Git, docker compose up, smoke checks»

> **Модуль**: 1 — Введение в тестирование и enterprise-системы
> **Тип**: Содержательная практика (воркшоп)
> **Артефакт**: №1 — Запуск СМП + smoke-тесты (6 баллов)
> **Проверка**: CI (сборка, контейнеры, smoke)
> **Длительность**: 1,5 астрономических часа (90 мин)
> **Формат**: Демонстрация → критерии → самостоятельная работа → разбор проблем
> **Дата**: 04.09.2026
> **Связь с лекциями**: [`lecture-01.md`](../slides/lecture-01.md) (СМП, слайд 10–12), [`lecture-02.md`](../slides/lecture-02.md) (модель данных, слайд 13)

---

## Цель практики

Запустить СМП на своей машине и убедиться, что все 11 сервисов + RabbitMQ работают, а сквозной прогон транзакции успешен. Это базовый smoke-тест: проверяется, что окружение готово к дальнейшей работе на семестр.

**Образовательная задача**: студент впервые сталкивается с реальным микросервисным проектом — Docker Compose, health-checks, сквозной путь транзакции. Это практическое воплощение того, о чём шла речь на лекциях 1–2.

---

## Структура практики (тайминг)

| Время | Блок | Назначение |
|:---:|---|---|
| 0:00–0:15 | Демонстрация | Преподаватель запускает СМП на экране |
| 0:15–0:20 | Критерии сдачи | Что должно работать для артефакта 1 |
| 0:20–0:80 | Самостоятельная работа | Студенты запускают СМП на своих машинах |
| 0:80–0:90 | Разбор проблем | Типовые ошибки, FAQ |

---

## Часть 1. Демонстрация (0:00–0:15)

Преподаватель на экране выполняет все шаги, которые студенты повторят самостоятельно:

1. Клонирует репозиторий СМП
2. Показывает структуру: `docker-compose.yaml`, `.env.example`, `scripts/smoke-test.sh`
3. Копирует `.env.example` → `.env`
4. Запускает `docker compose up -d`
5. Показывает `docker compose ps` — все 11 сервисов + RabbitMQ `Up`
6. Прогоняет health-check Gateway: `curl http://localhost:8080/health` — 200 OK
7. Прогоняет health-check Bin Lookup: `curl http://localhost:8096/health` — 200 OK
8. Прогоняет health-check Notification Service: `curl http://localhost:8097/health` — 200 OK
9. Открывает RabbitMQ Management UI: `http://localhost:15672` (логин: `smp`, пароль: `smp`)
10. Прогоняет `./scripts/smoke-test.sh` — `🎉 ALL CHECKS PASSED`
11. Открывает Dashboard: `http://localhost:3000`

**Скрипт для преподавателя:**

> Смотрите. Одна команда — `docker compose up -d` — и все 11 сервисов плюс RabbitMQ поднялись. Проверяем: `docker compose ps` — все `Up`. Health-check Gateway: `curl http://localhost:8080/health` — 200 OK. Новые сервисы: Bin Lookup — `curl http://localhost:8096/health` — 200 OK; Notification Service — `curl http://localhost:8097/health` — 200 OK. RabbitMQ Management UI открывается на `localhost:15672` — здесь можно видеть очереди, exchanges, dead-letter queues. Теперь smoke-тест: `./scripts/smoke-test.sh`. Видите — `ALL CHECKS PASSED`. Это значит: все сервисы отвечают, очереди работают, сквозной прогон транзакции успешен. Открываем Dashboard — `localhost:3000` — видим данные. Вот ваша цель на сегодня: повторить это на своей машине.

---

## Часть 2. Критерии сдачи (0:15–0:20)

**Чек-лист артефакта 1** (сверено с [`practic/docs/checklists.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/checklists.md) — раздел «Артефакт 1», и [`practic/README.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/README.md) — Быстрый старт):

- [ ] `docker compose up -d` поднимает все 11 сервисов + RabbitMQ без ошибок
- [ ] `curl http://localhost:{port}/health` каждого сервиса возвращает HTTP 200
- [ ] `curl http://localhost:8096/actuator/health` — Bin Lookup возвращает 200
- [ ] `curl http://localhost:8097/actuator/health` — Notification Service возвращает 200
- [ ] RabbitMQ Management UI доступен на `http://localhost:15672` (логин: `smp`, пароль: `smp`)
- [ ] `./scripts/smoke-test.sh` завершается `🎉 ALL CHECKS PASSED`
- [ ] Dashboard доступен на `http://localhost:3000`
- [ ] Генерация тестовых карт: `POST /api/cards/generate` отрабатывает
- [ ] Симулятор терминалов: `POST /api/simulator/terminal/run` отправляет транзакции

**Порты сервисов (шпаргалка):**

| Сервис | Порт | Health-check |
|--------|:---:|---|
| Gateway Service | 8080 | `curl http://localhost:8080/health` |
| Card Management | 8081 | `curl http://localhost:8081/health` |
| Switch / Router | 8082 | `curl http://localhost:8082/health` |
| Authorization | 8083 | `curl http://localhost:8083/health` |
| Terminal Simulator | 8085 | `curl http://localhost:8085/health` |
| Merchant Simulator | 8084 | `curl http://localhost:8084/health` |
| Transaction Logger | 8088 | `curl http://localhost:8088/health` |
| Bin Lookup | 8096 | `curl http://localhost:8096/actuator/health` |
| Notification Service | 8097 | `curl http://localhost:8097/actuator/health` |
| Web Dashboard | 3000 | браузер |
| RabbitMQ Management UI | 15672 | браузер |

**Текст для преподавателя:**

> Критерии сдачи. Первый: `docker compose up -d` поднимает все 11 сервисов и RabbitMQ без ошибок. Второй: health-check каждого сервиса возвращает 200. Порты — вот на этой шпаргалке. Третий: RabbitMQ Management UI открывается на `localhost:15672`. Четвёртый: smoke-тест завершается `ALL CHECKS PASSED`. Пятый: Dashboard открывается. Шестой: генерация карт работает. Седьмой: симулятор терминалов отправляет транзакции. Если все пункты зелёные — артефакт 1 сдан, 6 баллов ваших.
>
> Про сам механизм сдачи — куда пушить, как оформить Issue и что туда вставить — смотрите [`submission-guide.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/submission-guide.md). Коротко: работаем в своём репозитории `Practic{Имя}{Фамилия}`, а сдаёмся Issue в эталонном репозитории со ссылкой на свой Actions run.

---

## Часть 3. Самостоятельная работа (0:20–0:80)

### Шаг 0. Проверка предварительных требований

- [ ] Docker Desktop установлен и запущен
- [ ] Память Docker Desktop ≥ 4 ГБ
- [ ] Порты 3000, 8080–8097, 5432, 5672, 15672 свободны
- [ ] Git установлен

### Шаг 1. Клонирование своего репозитория

```bash
git clone https://github.com/<org>/Practic<Имя><Фамилия>.git
cd Practic<Имя><Фамилия>
```

### Шаг 2. Настройка окружения

```bash
cp .env.example .env
```

### Шаг 3. Запуск сервисов

```bash
docker compose up -d
```

> **Профиль `observability`:** Prometheus, Grafana, Loki, Promtail и autoscaler вынесены в отдельный compose-профиль и по умолчанию выключены. Для базовой практики достаточно `docker compose up -d` (11 сервисов + PostgreSQL + RabbitMQ). Полный стек с мониторингом запускается командой `docker compose --profile observability up -d` — он понадобится в модуле 8.

### Шаг 4. Проверка health-check (все 11 сервисов)

```bash
curl http://localhost:8080/health   # Gateway
curl http://localhost:8081/health   # Card Management
curl http://localhost:8082/health   # Switch
curl http://localhost:8083/health   # Authorization
curl http://localhost:8085/health   # Terminal Simulator
curl http://localhost:8084/health   # Merchant Simulator
curl http://localhost:8088/health   # Transaction Logger
curl http://localhost:8096/health   # Bin Lookup
curl http://localhost:8097/health   # Notification Service
```

RabbitMQ Management UI: `http://localhost:15672` (логин: `smp`, пароль: `smp`)

### Шаг 5. Smoke-тест

```bash
# Linux/Mac
./scripts/smoke-test.sh

# Windows
.\scripts\smoke-test.ps1
```

Ожидаемый результат: `🎉 ALL CHECKS PASSED`.

### Шаг 6. Генерация тестовых карт

```bash
curl -X POST http://localhost:8080/api/cards/generate \
  -H "Content-Type: application/json" \
  -d '{"count": 100, "bins": ["400000","400001","400002","400003","400004"]}'
```

### Шаг 7. Запуск симулятора терминалов

```bash
curl -X POST http://localhost:8080/api/simulator/terminal/run \
  -H "Content-Type: application/json" \
  -d '{"count": 50, "scenario": "normal"}'
```

### Шаг 8. Открытие Dashboard

Браузер: `http://localhost:3000`

### Быстрая проверка для куратора (одной командой)

```bash
docker compose up -d && sleep 10 && ./scripts/smoke-test.sh
```

Если вывод заканчивается строкой `🎉 ALL CHECKS PASSED` — все 11 сервисов работают, сквозной прогон транзакций успешен.

---

## Часть 4. Разбор типовых проблем (0:80–0:90)

| Проблема | Решение |
|----------|---------|
| Docker не установлен / не запущен | Установить Docker Desktop, запустить |
| Порт занят (8080, 3000 и т.д.) | `docker compose down`, освободить порт или изменить в `.env` |
| `docker compose up` падает с OOM | Увеличить память Docker Desktop до 4 ГБ |
| Health-check возвращает не 200 | `docker compose logs {service}`, проверить логи |
| Smoke-test падает на транзакции | Проверить, что карты сгенерированы (шаг 6) |
| Windows: `./scripts/smoke-test.sh` не работает | Использовать `.\scripts\smoke-test.ps1` или Git Bash |
| Dashboard не открывается | Проверить `docker compose ps`, контейнер dashboard должен быть Up |
| RabbitMQ не стартует | Проверить `docker compose logs rabbitmq`, порты 5672/15672 не заняты |
| Bin Lookup не отвечает | Проверить `docker compose logs bin-lookup`, порт 8096 не занят |
| Notification Service не отвечает | Проверить `docker compose logs notification-service`, порт 8097 не занят |
| Ошибка «port already in use» для 8096/8097 | Освободить порт или изменить `BIN_LOOKUP_PORT`/`NOTIFICATION_PORT` в `.env` |
| Ошибка аутентификации RabbitMQ UI | Проверить `.env`: `RABBITMQ_USER=smp`, `RABBITMQ_PASSWORD=smp` |
| `cp .env.example .env` не работает (Windows PowerShell) | `Copy-Item .env.example .env` |
| Git clone не работает | Проверить доступ к репозиторию, VPN при необходимости |

---

## Сдача артефакта

**Workflow сдачи**: см. [`practic/docs/submission-guide.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/submission-guide.md).

- Репозиторий: свой `Practic{Имя}{Фамилия}` (выдаёт куратор)
- Сдача через GitHub Issue в эталонном репозитории с label `practice-1` (ссылка на репозиторий + ссылка на Actions run)
- Проверка: CI (лёгкий — автоматически, тяжёлый smoke — вручную через «Practice Run» → practice-1)
- Баллы: 6 базовых + 5 бонусных за сдачу в дедлайн

**Что проверит CI (smoke, запускается вручную):**
1. Репозиторий клонируется
2. `.env` корректно создан из `.env.example`
3. `docker compose up -d` поднимает все 11 сервисов + RabbitMQ
4. Все health-check эндпоинты возвращают 200
5. `smoke-test.sh` завершается `ALL CHECKS PASSED`

> Тяжёлая проверка `smoke-tests` не гоняется на каждый push — она билдит docker-образы. Запустите её вручную перед сдачей: **Actions → Practice Run (heavy checks) → Run workflow → выбрать `1`**. Ссылку на зелёный прогон вставьте в Issue.

---

## Домашнее задание (до практики 2)

1. Пробежаться по архитектуре СМП: [`practic/docs/architecture.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/architecture.md)
2. Прочитать технические задания: [`tz/04-authorization.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/tz/04-authorization.md), [`tz/05-card-management.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/tz/05-card-management.md)
3. Сформулировать 3–5 пользовательских сценариев: успешная транзакция, отказ по разным причинам
4. Ознакомиться с чек-листами само-приёмки: [`practic/docs/checklists.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/checklists.md)

---

## Связанные документы

- [`practic/README.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/README.md) — концепция СМП, быстрый старт
- [`practic/docs/architecture.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/architecture.md) — архитектура и диаграммы
- [`practic/docs/checklists.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/checklists.md) — чек-листы само-приёмки
- [`practic/docs/git-workflow.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/git-workflow.md) — инструкция по Git
- [`practic/docs/submission-guide.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/submission-guide.md) — сдача артефактов
