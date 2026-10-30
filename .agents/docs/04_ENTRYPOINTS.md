# Точки входа

> Файлы, с которых начинается работа с репозиторием, и их потребители.
> В `Lectures` нет исполняемых точек входа — только документы.
> **Обновлять при изменении:** `COURSE_SYLLABUS.md`, `materials/`

## Обзор

Репозиторий не содержит CLI, HTTP API, очередей, UI и фоновых задач. Точки входа —
это документы, с которых разные потребители начинают работу: студент — с
силлабуса и FAQ, лектор — с каталога своего модуля, агент — с `AGENTS.md` и
`.agents/docs/`. Отдельная «точка входа» на уровне доставки — git-remote на GitHub.

## Детали

### Исполняемые точки входа

Отсутствуют. Подтверждено: нет `package.json`, `Makefile`, `.github/workflows/`,
Dockerfile, скриптов. Весь контент — статические Markdown/PDF/JPEG.

### Документные точки входа

| Тип | Путь | Потребитель | Что получает | Доступ |
|---|---|---|---|---|
| Силлабус | `COURSE_SYLLABUS.md` | Студент, лектор, методист | Параметры курса, модули, оценивание 40+50+10, расписание | Публичный GitHub |
| Публичное описание | `materials/admin/DocsForItmoStudents/TestingFor.md` | Студент, вуз | Компактное описание курса, программа 8 модулей, стек | Публичный GitHub |
| FAQ | `materials/admin/faq-students.md` | Студент | Дедлайны, порядок сдачи, порты сервисов, политика LLM | Публичный GitHub |
| Лекция | `materials/module-NN-*/slides/lecture-NN.md` | Студент, лектор | Слайды и speaker notes | Публичный GitHub |
| PDF лекции | `materials/module-01-intro/slides/lecture-0N.pdf` | Студент | Готовый экспорт лекции | Публичный GitHub |
| Иллюстрации | `materials/module-01-intro/slides/Pictires/*.jpeg` | Лектор | Фоны и схемы для слайдов | Публичный GitHub |
| Практика | `materials/module-NN-*/practice/practice-NN.md` | Студент, преподаватель | Сценарий, критерии, чек-лист сдачи | Публичный GitHub |
| Шаблон артефакта | `materials/artifacts/artifact-N-*.md` | Студент | Требования и структура сдаваемой работы | Публичный GitHub |
| РПД | `testing-course-internal/plans/РПД_Тестирование_ПО_ИТМО.md` (актуальный источник); локальная копия `materials/admin/РПД_Тестирование_ПО_ИТМО.md` (untracked, не публикуется) | Методист, вуз | Рабочая программа дисциплины | Приватный GitHub / локально |
| Ревью контента (удалён 15.09.2026) | `materials/admin/review-module-02-open-questions.md` — файла в репозитории нет; копия вне репозитория (`C:\GIT\github-publish\.local-backup-20260915\`); решения — `SlidevSlides/plans/План_синхронизации_модуля_02.md` | Лекторы, методист | История ревью модуля 2 | — |
| Индекс для агента | `AGENTS.md` | AI-агент | Навигация по `.agents/docs/` | Публичный GitHub |

### Точка входа доставки

| Тип | Значение | Примечание |
|---|---|---|
| Git remote | `origin https://github.com/LektonSoftwareTestingCourse/Lectures.git` | Ветка `main` — основная и единственная |
| Протокол | HTTPS | Аутентификация — стандартная для GitHub |
| Потребители | студенты, лекторы, соседние репозитории | Клонирование или веб-просмотр |

### Фактическое наполнение по модулям

| Модуль | Лекции | Практики | Прочее |
|---|---|---|---|
| 1 Введение | `lecture-01.md`, `lecture-02.md` (+PDF) | `practice-01.md`, `practice-02.md` | `Pictires/` 5 JPEG |
| 2 Тест-дизайн | `lecture-03.md`, `lecture-04.md` (+PDF) | `practice-01.md` | `Pictires/` 11 файлов; влито PR #2 — см. Q-2 |
| 3 Пирамида и CI/CD | `lecture-05.md`, `lecture-06.md` | нет | — |
| 4 Unit-тесты | `lecture-01.md`, `lecture-02.md` | `practice-01.md`, `practice-02.md` | `demo/` пуст |
| 5 API-тесты | `lecture-09.md`, `lecture-10.md` | нет | `demo/` пуст |
| 6 Интеграционные | `lecture-01.md`, `lecture-02.md` | `practice-01.md`, `practice-02.md` | `demo/` пуст |
| 7 E2E | нет | нет | только `.gitkeep` |
| 8 CI/CD и стратегия | нет | нет | только `.gitkeep` |

### Точки входа в соседних репозиториях (внешние)

| Репозиторий | Файл/ссылка | Роль |
|---|---|---|
| `Practic` | `docs/submission-guide.md` | Порядок и дедлайны сдачи |
| `Practic` | `docs/git-workflow.md` | Инструкция по Git для студентов |
| `Practic` | `README.md` | Карта сервисов и портов СМП |
| `Practic` | `.github/workflows/ci.yml` | CI-пайплайн эталонного проекта |
| `testing-course-internal` | `materials/llm/skills/skill-1..N` | LLM-рубрики проверки артефактов |
| `SlidevSlides` | `slides/module-02/lecture-03.md`, `lecture-04.md` | Слайды модуля 2 |

## Связи

- Потоки использования точек входа — `03_DATA_FLOW.md`.
- Конвенции оформления входа — `06_COURSE_STRUCTURE.md`.
- Открытые вопросы по отсутствующим входам — `00_OPEN_QUESTIONS.md`.
