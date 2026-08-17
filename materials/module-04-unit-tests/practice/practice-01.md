# Practice 1: «JUnit 5: assertions, lifecycle, fixtures»

> **Модуль**: 4 — Unit-тестирование на Java / JUnit 5
> **Тип**: Содержательная практика (воркшоп)
> **Артефакт**: №3 — Unit-тесты JUnit 5 (7 баллов)
> **Проверка**: CI (сборка, прогон тестов, JaCoCo coverage)
> **Длительность**: 1,5 астрономических часа (90 мин)
> **Формат**: Демонстрация → критерии → самостоятельная работа → разбор проблем
> **Дата**: 03.08.2026
> **Связь с лекциями**: [`lecture-01.md`](../slides/lecture-01.md) (Принципы FIRST, AAA, test doubles)

---

## Цель практики

Настроить в проекте СМП базовое окружение для Unit-тестирования (JUnit 5 + Mockito) и написать первые изолированные тесты для бизнес-логики сервиса Gateway. Настроить первый CI-пайплайн в GitHub Actions для автоматического прогона тестов.

**Образовательная задача**: студент применяет паттерн AAA, принципы FIRST и аннотации жизненного цикла JUnit 5 на практике, изолируя зависимости с помощью моков.

---

## Структура практики (тайминг)

| Время | Блок | Назначение |
|:---:|---|---|
| 0:00–0:15 | Демонстрация | Преподаватель пишет первый unit-тест на экране |
| 0:15–0:20 | Критерии сдачи | Что должно работать для артефакта 3 |
| 0:20–0:80 | Самостоятельная работа | Студенты пишут тесты для Gateway |
| 0:80–0:90 | Разбор проблем | Типовые ошибки мокирования, настройки Maven |

---

## Часть 1. Демонстрация (0:00–0:15)

Преподаватель на экране:
1. Открывает проект СМП (сервис `gateway`) в IntelliJ IDEA.
2. Добавляет зависимости `junit-jupiter` и `mockito-core` в `pom.xml` (или `build.gradle`).
3. Создает класс `GatewayServiceTest` в `src/test/java`.
4. Пишет тест с использованием паттерна AAA (Arrange-Act-Assert).
5. Демонстрирует аннотации `@BeforeEach`, `@AfterEach` для управления фикстурами.
6. Показывает, как замокировать зависимость (например, `SwitchClient`) с помощью `@Mock` и `@InjectMocks`.
7. Запускает тест локально, показывает зелёную полосу.
8. Создает файл `.github/workflows/tests.yml` для запуска тестов в CI.

**Скрипт для преподавателя:**

> Смотрите. Мы добавили JUnit 5 в проект. Создаём тестовый класс. Обратите внимание на структуру: мы настраиваем окружение в `@BeforeEach`, чтобы тесты были изолированы (принцип FIRST). В блоке Arrange мы задаём входные данные. В Act — вызываем метод. В Assert — проверяем результат. Заметьте, как мы отсекли реальный вызов в Switch с помощью мока `@Mock` — теперь наш тест выполняется за миллисекунды. Теперь добавим простой workflow в GitHub Actions, чтобы эти тесты запускались при каждом пуше.

---

## Часть 2. Критерии сдачи (0:15–0:20)

**Чек-лист артефакта 3 (базовая часть):**

- [ ] В проект добавлены зависимости JUnit 5 и Mockito
- [ ] Создан класс `GatewayServiceTest` (или аналогичный)
- [ ] Использован паттерн AAA (визуально разделён на блоки)
- [ ] Использованы аннотации жизненного цикла (`@BeforeEach`)
- [ ] Зависимости замокированы (`@Mock`, `@InjectMocks`)
- [ ] Тесты успешно запускаются локально (`mvn test`)
- [ ] Настроен базовый CI-пайплайн в GitHub Actions

**Текст для преподавателя:**

> Критерии сдачи. Вы должны настроить проект для тестирования. Написать хотя бы 3-5 тестов для Gateway, используя правильную структуру AAA. Обязательно использовать моки для внешних зависимостей, чтобы тесты были быстрыми. И настроить CI, чтобы я видел, что ваши тесты проходят в GitHub Actions.

---

## Часть 3. Самостоятельная работа (0:20–0:80)

### Шаг 1. Настройка зависимостей

В `pom.xml` сервиса Gateway добавьте:
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

### Шаг 2. Написание первого теста

1. Создайте класс `GatewayServiceTest` в `src/test/java/com/processing/gateway/service`.
2. Аннотируйте класс `@ExtendWith(MockitoExtension.class)`.
3. Создайте моки для зависимостей (например, `SwitchClient`).
4. Напишите тест `shouldForwardTransactionWhenValidRequest()`.
5. Используйте `when(switchClient.send(any())).thenReturn(successResponse)`.
6. Проверьте результат с помощью `assertThat(result).isEqualTo(...)`.

### Шаг 3. Настройка CI (GitHub Actions)

1. В корне репозитория создайте `.github/workflows/tests.yml`.
2. Настройте job на `ubuntu-latest` с JDK 21.
3. Добавьте шаг `mvn -B test --file gateway/pom.xml`.

---

## Часть 4. Разбор типовых проблем (0:80–0:90)

| Проблема | Решение |
|----------|---------|
| `UnnecessaryStubbingException` | Вы настраиваете мок в `@BeforeEach`, но не во всех тестах он используется. Перенесите настройку в конкретный тест. |
| `NullPointerException` в тесте | Забыли аннотацию `@ExtendWith(MockitoExtension.class)` или `@Mock` над полем. |
| CI падает с `Could not find artifact` | Убедитесь, что Maven-модули собираются в правильном порядке, или используйте корневой `pom.xml` для запуска тестов. |
| Тесты не запускаются | Класс должен оканчиваться на `Test`, а метод иметь аннотацию `@Test`. |

---

## Сдача артефакта

- Ветка: `practice-3-{surname}`
- Сдача через GitHub Issue с label `practice-3`
- Проверка: CI-автопроверка (сборка, прогон тестов)

---

## Связанные документы

- [`practic/docs/git-workflow.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/git-workflow.md) — инструкция по Git
- [`practic/docs/submission-guide.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/submission-guide.md) — сдача артефактов