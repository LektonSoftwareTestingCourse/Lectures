# Practice 2: «Интеграционные тесты для Logger и Authorization. Отладка»

> **Модуль**: 6 — Интеграционное тестирование с БД в контейнерах
> **Тип**: Содержательная практика (воркшоп)
> **Артефакт**: №4 — API и интеграционные тесты (7 баллов)
> **Проверка**: CI (сборка, прогон тестов, Testcontainers)
> **Длительность**: 1,5 астрономических часа (90 мин)
> **Формат**: Демонстрация → критерии → самостоятельная работа → разбор проблем
> **Дата**: 03.08.2026
> **Связь с лекциями**: [`lecture-02.md`](../slides/lecture-02.md) (Особенности интеграционного тестирования)

---

## Цель практики

Расширить интеграционные тесты для сервисов Transaction Logger и Authorization. Научиться управлять тестовыми данными (Fresh Setup), тестировать асинхронные процессы (RabbitMQ) с помощью Awaitility, и отлаживать Testcontainers в CI.

**Образовательная задача**: студент осваивает паттерны изоляции интеграционных тестов, учится работать с Eventual Consistency и избегать хрупких (flaky) тестов.

---

## Структура практики (тайминг)

| Время | Блок | Назначение |
|:---:|---|---|
| 0:00–0:15 | Демонстрация | Преподаватель пишет асинхронный тест с Awaitility |
| 0:15–0:20 | Критерии сдачи | Что должно работать для артефакта 4 |
| 0:20–0:80 | Самостоятельная работа | Студенты пишут тесты для Logger и Authorization |
| 0:80–0:90 | Разбор проблем | Flaky тесты, проблемы со временем в CI |

---

## Часть 1. Демонстрация (0:00–0:15)

Преподаватель на экране:
1. Открывает сервис `transaction-logger`.
2. Показывает, как логгер асинхронно сохраняет транзакции из RabbitMQ в БД.
3. Добавляет зависимость `awaitility` в `pom.xml`.
4. Создает `TransactionLoggerIT`.
5. Настраивает `@Container` для PostgreSQL и RabbitMQ (используя `@ServiceConnection`).
6. Пишет тест: публикует сообщение в RabbitMQ, затем с помощью Awaitility ждёт, пока запись не появится в БД.

**Скрипт для преподавателя:**

> Смотрите. Switch отправляет сообщение в очередь, и Logger его обрабатывает. Это асинхронно. База обновится не сразу. Если мы пойдём в БД сразу после отправки — тест упадёт. Мы используем Awaitility: пишем `await().atMost(5, SECONDS).until(() -> repository.count() == 1)`. Библиотека будет опрашивать базу каждые 100 миллисекунд. Как только лог появится — тест пойдёт дальше. Никаких `Thread.sleep()`. Это защищает нас от flaky-тестов.

---

## Часть 2. Критерии сдачи (0:15–0:20)

**Чек-лист артефакта 4 (расширенная часть):**

- [ ] Написаны интеграционные тесты для Transaction Logger
- [ ] Использована библиотека Awaitility для асинхронных проверок
- [ ] Настроен контейнер RabbitMQ в Testcontainers
- [ ] Тесты корректно очищают данные между прогонами (Fresh Setup / `@Transactional` / TRUNCATE)
- [ ] Использован паттерн Singleton Container для оптимизации скорости
- [ ] CI-пайплайн успешно прогоняет все тесты

**Текст для преподавателя:**

> Критерии сдачи. Вы должны написать интеграционный тест для Logger, который проверяет асинхронное сохранение в БД. Обязательно используйте Awaitility, никаких `sleep`. Настройте изоляцию: каждый тест должен работать с чистой базой. И используйте паттерн Singleton Container, чтобы не поднимать базу для каждого теста заново — иначе CI будет идти вечность.

---

## Часть 3. Самостоятельная работа (0:20–0:80)

### Шаг 1. Настройка Singleton Container

Создайте базовый класс для интеграционных тестов, чтобы переиспользовать контейнеры:
```java
public abstract class BaseIT {
    @Container
    @ServiceConnection
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @Container
    @ServiceConnection
    static RabbitMQContainer rabbitMQ = new RabbitMQContainer("rabbitmq:3.13-management");
}
```

### Шаг 2. Тестирование Transaction Logger

1. Создайте `TransactionLoggerIT extends BaseIT`.
2. Внедрите `RabbitTemplate` и `TransactionLogRepository`.
3. В `@Test` отправьте сообщение в очередь RabbitMQ.
4. Используйте Awaitility:
```java
await().atMost(5, SECONDS).untilAsserted(() -> {
    assertThat(repository.findAll()).hasSize(1);
});
```

### Шаг 3. Настройка изоляции (Fresh Setup)

1. Добавьте `@Transactional` на уровне тестового класса (если это срез `@DataJpaTest`).
2. Или используйте `@Sql(scripts = "/cleanup.sql", executionPhase = BEFORE_TEST_METHOD)` для TRUNCATE.
3. Убедитесь, что тесты можно запускать в любом порядке.

---

## Часть 4. Разбор типовых проблем (0:80–0:90)

| Проблема | Решение |
|----------|---------|
| `ConditionTimeoutException` | Сообщение не долетело до БД. Проверьте Routing Key, Exchange и Queue в RabbitMQ. |
| Тесты падают только в CI | Race condition. Увеличьте `atMost` для Awaitility. Проверьте, что CI не перегружен. |
| `IllegalStateException: shared container...` | Проблема со static полями. Убедитесь, что контейнеры `static` и `@Container` стоит над полем базового класса. |
| Данные остаются между тестами | `@Transactional` не помогает для асинхронных Consumer. Используйте `TRUNCATE` в `@AfterEach`. |
| `Docker daemon is not responding` | Проверьте ресурсы Docker. 2 контейнера (Postgres + RabbitMQ) требуют минимум 2 ГБ ОЗУ. |

---

## Сдача артефакта

- Ветка: `practice-4-{surname}`
- Сдача через GitHub Issue с label `practice-4`
- Проверка: CI-автопроверка (сборка, Testcontainers, RabbitMQ, прогон IT-тестов)

---

## Связанные документы

- [`practic/docs/git-workflow.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/git-workflow.md) — инструкция по Git
- [`practic/docs/submission-guide.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/submission-guide.md) — сдача артефактов