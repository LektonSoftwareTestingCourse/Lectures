# Practice 1: «Поднимаем PostgreSQL и RabbitMQ в контейнерах»

> **Модуль**: 6 — Интеграционное тестирование с БД в контейнерах
> **Тип**: Содержательная практика (воркшоп)
> **Артефакт**: №4 — API и интеграционные тесты (7 баллов)
> **Проверка**: CI (сборка, прогон тестов, Testcontainers)
> **Длительность**: 1,5 астрономических часа (90 мин)
> **Формат**: Демонстрация → критерии → самостоятельная работа → разбор проблем
> **Дата**: 03.08.2026
> **Связь с лекциями**: [`lecture-01.md`](../slides/lecture-01.md) (БД, очереди, внешние сервисы)

---

## Цель практики

Настроить в проекте СМП интеграционное тестирование с использованием библиотеки Testcontainers. Поднять реальную базу данных PostgreSQL 16 в Docker-контейнере прямо из JUnit 5 теста. Проверить корректность работы репозиториев сервиса Card Management (миграции Flyway, SELECT/UPDATE).

**Образовательная задача**: студент учится отходить от in-memory баз (H2) и моков, переходя к реальным СУБД в тестах, управляя жизненным циклом контейнеров через Java-код.

---

## Структура практики (тайминг)

| Время | Блок | Назначение |
|:---:|---|---|
| 0:00–0:15 | Демонстрация | Преподаватель настраивает Testcontainers и пишет первый тест |
| 0:15–0:20 | Критерии сдачи | Что должно работать для артефакта 4 |
| 0:20–0:80 | Самостоятельная работа | Студенты пишут интеграционные тесты для Card Management |
| 0:80–0:90 | Разбор проблем | Ошибки скачивания образов, настройки JDBC URL |

---

## Часть 1. Демонстрация (0:00–0:15)

Преподаватель на экране:
1. Открывает сервис `card-management` в IDE.
2. Добавляет зависимости `testcontainers`, `postgresql`, `testcontainers-junit-jupiter` в `pom.xml`.
3. Создает класс `CardManagementRepositoryIT` (оканчивается на `IT` для плагина Failsafe).
4. Настраивает `PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");`.
5. Показывает, как Spring Boot (`@ServiceConnection` или `@DynamicPropertySource`) подхватывает URL контейнера.
6. Пишет тест: сохраняет карту в БД, затем делает `findById`, проверяет статус.
7. Запускает тест, показывает логи Testcontainers (скачивание образа, старт БД, прогон теста, остановка).

**Скрипт для преподавателя:**

> Смотрите. Мы добавили Testcontainers. Создаём контейнер `postgres:16`. Spring Boot 3 умеет сам подставлять URL базы в свойства через `@ServiceConnection`. Когда запускаем тест, Testcontainers обращается к Docker, поднимает чистую базу, применяет миграции Flyway, прогоняет тест и убивает контейнер. Мы тестируем реальный SQL, реальный PostgreSQL. Никаких H2. Никаких моков репозиториев.

---

## Часть 2. Критерии сдачи (0:15–0:20)

**Чек-лист артефакта 4 (базовая часть):**

- [ ] Добавлены зависимости Testcontainers в проект
- [ ] Создан класс `*IT.java` для интеграционных тестов
- [ ] Настроен `PostgreSQLContainer` (версия 16)
- [ ] Контейнер корректно связан со Spring Context (`@ServiceConnection` или `@DynamicPropertySource`)
- [ ] Тест проверяет запись в БД (`save`) и чтение (`findById`)
- [ ] Тест успешно запускается локально (`mvn verify`)
- [ ] Контейнер корректно стартует и в CI (GitHub Actions)

**Текст для преподавателя:**

> Критерии сдачи. Вы должны добавить Testcontainers в Card Management. Написать интеграционный тест, который поднимает реальный PostgreSQL 16, сохраняет карту и читает её обратно. Убедитесь, что тест запускается через `mvn verify` (используем Failsafe плагин для `*IT` классов), а не `mvn test`. И проверьте, что в GitHub Actions Docker доступен и тест проходит.

---

## Часть 3. Самостоятельная работа (0:20–0:80)

### Шаг 1. Добавление зависимостей в `card-management/pom.xml`

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

### Шаг 2. Настройка контейнера в тесте

1. Создайте `CardManagementRepositoryIT`.
2. Аннотируйте класс `@SpringBootTest` и `@Testcontainers`.
3. Добавьте поле:
```java
@Container
@ServiceConnection
static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
```

### Шаг 3. Написание интеграционного теста

1. Внедрите `CardRepository` в тест.
2. В `@Test` создайте карту с статусом `ACTIVE`.
3. Вызовите `repository.save(card)`.
4. Вызовите `repository.findById(card.getId())`.
5. Убедитесь, что возвращенная карта имеет статус `ACTIVE` (используйте AssertJ `assertThat(...).isEqualTo(...)`).

---

## Часть 4. Разбор типовых проблем (0:80–0:90)

| Проблема | Решение |
|----------|---------|
| `Could not find a valid Docker environment` | Запустите Docker Desktop. Убедитесь, что IDE имеет права доступа к сокету Docker. |
| Тест висит на старте | Testcontainers скачивает образ. В первый раз это может занять время. Проверьте интернет-соединение. |
| `Connection refused` | Spring не видит порт контейнера. Убедитесь, что используете `@ServiceConnection` (Spring Boot 3.1+) или `@DynamicPropertySource`. |
| CI падает с `Docker not found` | В GitHub Actions Docker доступен по умолчанию, но убедитесь, что job запускается на `ubuntu-latest`. |
| `No tests found` | Класс называется `*IT`, а плагин Surefire ищет `*Test`. Добавьте Failsafe плагин для `*IT` классов. |

---

## Сдача артефакта

- Ветка: `practice-4-{surname}`
- Сдача через GitHub Issue с label `practice-4`
- Проверка: CI-автопроверка (сборка, Testcontainers, прогон IT-тестов)

---

## Связанные документы

- [`practic/docs/git-workflow.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/git-workflow.md) — инструкция по Git
- [`practic/docs/submission-guide.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/submission-guide.md) — сдача артефактов