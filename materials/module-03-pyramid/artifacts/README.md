# Артефакты практики «Пирамида тестирования» (модуль 3)

> **Сдача**: только индивидуальная защита. Артефакты — рабочий материал для защиты.
> **Акцент**: white-box — каждая диаграмма и уровень тест-кейса обосновываются **кодом**.

## Что сдаёте

В репозитории студента в `docs/practice-3-pyramid/`:

| Что | Куда |
|---|---|
| 6 диаграмм (PlantUML или mermaid, делаете сами) | `docs/practice-3-pyramid/architecture/` |
| Тест-кейсы + анализ пирамиды | `docs/practice-3-pyramid/test-cases.md` |

> Диаграммы студент строит **сам** — готовых скелетов/заглушек нет. Ниже — только текстовые требования к каждой.

## Что требуется от каждой диаграммы

| № | Диаграмма | Что показать | Чего не хватает в проекте |
|---|---|---|---|
| А1 | C4 Context | внешние акторы (терминал, мерчант, пользователь дашборда, внешний BIN Lookup) + протоколы входа | отсутствует |
| А2 | C4 Container | 11 сервисов + PostgreSQL + RabbitMQ; **разметка sync (HTTP/JDBC) vs async (AMQP)** + exchange/queue | есть, но не размечены sync/async, не сверены с docker-compose |
| А3 | C4 Component (Authorization, Card-Management) | controller → service → client/repository; где HTTP-клиенты, JDBC, outbox | отсутствует |
| А4 | Sequence | happy path + declined + reversal (mti 0400); где sync, где async | есть только happy path |
| А5 | RabbitMQ-топология | exchange → queue → consumer, DLX/DLQ, publisher confirms (2s), retry (3, TTL 60s) | описана текстом, не визуализирована |
| А6 | State-transition | Card.status + жизненный цикл Transaction | отсутствует |

## Источники для верификации (читать код)

- [`services/`](../../../Practic-SMP/services/) — **источник истины**;
- [`docs/architecture.md`](../../../Practic-SMP/docs/architecture.md) — сверка/достройка;
- [`docs/architecture-diagram.puml`](../../../Practic-SMP/docs/architecture-diagram.puml) — верифицируемая Container-схема;
- [`docs/api-spec.md`](../../../Practic-SMP/docs/api-spec.md) — API-контракты;
- [`tz/`](../../../Practic-SMP/tz/) — технические задания.

## Структура тест-кейса (модуль 2)

`ID | Связанное требование | Уровень пирамиды | Вид | Предусловие | Шаги | Ожидаемый результат | Источник`

Колонка «Уровень пирамиды» (`unit` / `api` / `integration` / `e2e`) — обязательное дополнение к формату модуля 2.

## Ограничение на LLM

Запрещено генерировать диаграммы и тест-кейсы через LLM. Допустим справочный запрос по синтаксису PlantUML/mermaid — после того, как вы сами определили, что рисовать.

## Задание и инструктаж

- Задание: [`../practice/practice-01.md`](../practice/practice-01.md)
- Разбор по диаграммам: [`../practice/practice-01-guide.md`](../practice/practice-01-guide.md)
