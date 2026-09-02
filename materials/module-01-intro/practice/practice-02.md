# Practice 2: «Проходная 1 — приём артефакта 1 + разбор архитектуры СМП, карта требований»

> **Модуль**: 1 — Введение в тестирование и enterprise-системы
> **Тип**: Проходная практика (приёмка-first + опциональный содержательный блок)
> **Артефакт**: приём Артефакта 1 — Запуск СМП + smoke-тесты (6 баллов)
> **Проверка**: CI зелёный **или** локальный прогон smoke по чек-листу
> **Длительность**: 1,5 астрономических часа (90 мин)
> **Формат**: Вводная (≤10 мин) → приёмка артефакта 1 → (опц.) разбор архитектуры + карта требований (≤30 мин) → Q&A/ДЗ
> **Дата**: 11.09.2026
> **Связь с лекциями**: [`lecture-01.md`](../slides/lecture-01.md) (архитектура СМП, слайд 11–12), [`lecture-02.md`](../slides/lecture-02.md) (модель данных, классы эквивалентности)
> **Дедлайн артефакта 1**: 18.09.2026 (неделя 3)

---

## Цель практики

Принять у студентов Артефакт 1 «Запуск СМП + smoke-тесты» по единому чек-листу и, если приёмка не растянулась, провести опциональный разбор архитектуры СМП с построением «карты требований» — мостика к модулю 2 (тест-дизайн) и Артефакту 2.

**Образовательная задача**: студент не только «закрывает» артефакт, но и связывает запуск системы с её архитектурой и требованиями — учится отвечать на вопрос «что мы тестируем и почему именно это».

---

## Структура практики (тайминг)

| Время | Блок | Назначение |
|:---:|---|---|
| 0:00–0:10 | Вводная | Статус потока, правила приёма, дедлайн |
| 0:10–0:50 | Приёмка артефакта 1 | Основная работа: приём по чек-листу |
| 0:50–0:80 | (опц.) Разбор архитектуры СМП + карта требований | Резиновый блок, дропается при затянувшейся приёмке |
| 0:80–0:90 | Q&A + ДЗ | План до 18.09, анонс модуля 2 |

> **Формат «приёмка-first»** (согласован 31.08.2026): рассказ только первые 5–10 минут. Дальше — приёмка и Q&A. Опциональный блок архитектуры сжимается до 15 минут или дропается без потери DoD.

---

## Часть 1. Вводная (0:00–0:10)

**Что на экране:**
- Статус потока: список Issue'ов с label `practice-1`, CI-статусы веток студентов;
- Правила приёма (из [`submission-guide.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/submission-guide.md));
- Дедлайн Артефакта 1 — **18.09.2026**.

**Скрипт для преподавателя:**

> На прошлой практике вы запускали СМП и гоняли smoke. Сегодня — проходная: я и 1–2 ассистента параллельно принимаем Артефакт 1 по чек-листу. Правила простые: либо у вас в ветке зелёный CI, либо мы локально прогоняем smoke по чек-листу. Дедлайн артефакта 1 — 18 сентября. У кого не заработало на прошлой практике — сегодня доделываем, все вопросы — в общий документ, ассистенты отвечают там же.

---

## Часть 2. Приёмка артефакта 1 (0:10–0:50)

### Как организована приёмка

- Ведущий + 1–2 ассистента принимают **параллельно** по единому чек-листу.
- Критерий сдачи: **CI зелёный** в ветке студента **или** **локальный прогон smoke** по чек-листу ниже.
- Типовые проблемы фиксируются в **общем документе проблем** — ассистенты отвечают там же, не прерывая поток.
- Итог по каждому студенту — отметка «сдано / доработать».

### Чек-лист приёмника артефакта 1

Сверено с [`scripts/smoke-test.sh`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/scripts/smoke-test.sh) — это источник истины по порядку проверок.

- [ ] `docker compose up -d` поднимает все 11 сервисов + PostgreSQL + RabbitMQ без ошибок
- [ ] `docker compose ps` — все контейнеры `Up` (healthy)
- [ ] Health-check Gateway: `curl http://localhost:8080/health` → 200
- [ ] Health-check Card Management: `curl http://localhost:8081/health` → 200
- [ ] Health-check Switch: `curl http://localhost:8082/health` → 200
- [ ] Health-check Authorization: `curl http://localhost:8083/health` → 200
- [ ] Health-check Terminal Simulator: `curl http://localhost:8085/health` → 200
- [ ] Health-check Merchant Simulator: `curl http://localhost:8084/health` → 200
- [ ] Health-check Transaction Logger: `curl http://localhost:8088/health` → 200
- [ ] Health-check Bin Lookup: `curl http://localhost:8096/actuator/health` → 200
- [ ] Health-check Notification Service: `curl http://localhost:8097/actuator/health` → 200
- [ ] RabbitMQ Management UI доступен: `http://localhost:15672` (логин `smp`, пароль `smp`)
- [ ] Web Dashboard доступен: `http://localhost:3000`
- [ ] `./scripts/smoke-test.sh` завершается `🎉 ALL CHECKS PASSED`
- [ ] Генерация карт: `POST /api/cards/generate` отрабатывает (≥ 20 карт)
- [ ] Симулятор терминалов: `POST /api/simulator/terminal/run` отправляет транзакции (50 submitted)

### Порты сервисов (шпаргалка)

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

---

## Часть 3. (Опционально, ≤30 мин) Разбор архитектуры СМП + карта требований

> **Блок резиновый:** выполняется, только если приёмка завершилась раньше 0:50. При затянувшейся приёмке дропается или сжимается до 15 минут. DoD занятия от этого блока не зависит.

### 3.1. Разбор архитектуры (0:50–0:65)

**Что на экране:**
- [`architecture.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/architecture.md) — sequence-диаграмма пути транзакции;
- RabbitMQ Management UI (`http://localhost:15672`) — очереди `transaction-log`, `card-notifications`, DLQ.

**Скрипт:**

> Смотрим на sequence-диаграмму. Терминал → Gateway → Switch → Authorization → Card Management и Bin Lookup. Ключевой момент — резервирование: между GET карты и POST /reserve возможна гонка. Откройте RabbitMQ Management UI: вот очередь `transaction-log` — сюда Switch публикует транзакции, а Logger потребляет. Это eventual consistency: запись в лог происходит не мгновенно.

**Вопросы аудитории:**
- Какой сервис здесь самый нагруженный? *(Gateway/Switch)*
- Какая самая критичная точка отказа? *(Authorization — единственный, кто принимает решение по деньгам)*

### 3.2. Карта требований (0:65–0:80)

**Терминологическая вводная (≤10 мин):** формальная теория классов эквивалентности — модуль 2 (Ольга, 17.09), но термины уже звучали на лекции 2 (10.09). Здесь работаем с «сырым» материалом: поля, статусы, сценарии — как заготовка к Артефакту 2.

**Что делаем:** совместно строим 3–5 строк «карты требований» по ТЗ [`04-authorization.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/tz/04-authorization.md) и [`05-card-management.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/tz/05-card-management.md).

**Шаблон карты требований:**

| Требование (источник) | Класс эквивалентности / граница | Будущий тест |
|---|---|---|
| Проверка статуса карты (`04-authorization.md`, п. 2) | status: ACTIVE / INACTIVE / BLOCKED / EXPIRED | Авторизация с картой в каждом статусе → ожидаемый decline |
| Проверка дневного лимита (`04-authorization.md`, п. 4) | amount ≤ dailyLimit / amount = dailyLimit / amount > dailyLimit | Транзакция на границу лимита → APPROVED / DECLINED |
| Проверка баланса (`04-authorization.md`, п. 6) | amount ≤ availableBalance / amount > availableBalance | Транзакция с недостаточным балансом → INSUFFICIENT_FUNDS |
| Резервирование (`05-card-management.md`, п. 5) | availableBalance −= amount | После APPROVED баланс карты уменьшился на сумму |
| Генерация карт (`05-card-management.md`, п. 3) | count карт, распределение по BIN, статусы 95/3/2 | POST /api/cards/generate → count карт, валидный PAN (Луна) |

**Домашняя заготовка к артефакту 2** — студенты продолжают карту самостоятельно (см. ДЗ ниже).

---

## Часть 4. Q&A + ДЗ (0:80–0:90)

**Что на экране:** план до дедлайна 18.09; анонс модуля 2.

**Домашнее задание:**
1. Дочитать ТЗ [`04-authorization.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/tz/04-authorization.md) и [`05-card-management.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/tz/05-card-management.md).
2. Сформулировать 3–5 пользовательских сценариев (успешная транзакция + отказы по разным причинам) — синхронизировано с ДЗ практики 1.
3. Продолжить карту требований из Части 3.2 ещё на 3–5 строк.

---

## DoD занятия

Преподаватель на занятие имеет:
1. **Чек-лист приёмника артефакта 1** (CI зелёный **или** локальный прогон smoke) — Часть 2.
2. **1–2 ассистента**, знающих чек-лист.
3. **Шаблон карты требований** (опциональный блок) — Часть 3.2.
4. **Общий документ проблем** для фиксации типовых ошибок.
5. **Сценарий** в `practice-02.md`: вводная ≤10 мин → приёмка → (опц.) архитектура ≤30 мин → Q&A/ДЗ.

---

## Связанные документы

- [`practic/docs/architecture.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/architecture.md) — архитектура, sequence-диаграмма, порты
- [`practic/tz/04-authorization.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/tz/04-authorization.md) — ТЗ Authorization
- [`practic/tz/05-card-management.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/tz/05-card-management.md) — ТЗ Card Management
- [`practic/docs/submission-guide.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/submission-guide.md) — workflow сдачи через Issue
- [`practic/docs/checklists.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/checklists.md) — чек-листы само-приёмки
- [`practic/scripts/smoke-test.sh`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/scripts/smoke-test.sh) — источник истины по порядку проверок
- [`practice-01.md`](practice-01.md) — содержательная практика 1 (запуск СМП + smoke)
