# Шаблон артефакта 1 — Запуск СМП + smoke-тесты

> **Практика**: 1 (содержательная, 6 баллов)
> **Проверка**: CI-автопроверка (`smoke-tests`)
> **Дедлайн**: конец недели 3

## Что сдаёте

Подтверждение, что вы развернули СМП и прогнали smoke-проверки. Кодового артефакта нет — сдача через ветку + Issue с label `practice-1`.

## Чек-лист приёмки (заполните `[x]`)

- [ ] `docker compose up -d` поднимает 11 сервисов + PostgreSQL + RabbitMQ без ошибок
- [ ] `docker compose ps` — все контейнеры `Up` (healthy)
- [ ] Все health-check эндпоинты возвращают HTTP 200 (порты 8080–8097, см. [`checklists.md`](https://github.com/LektonSoftwareTestingCourse/Practic/blob/main/docs/checklists.md))
- [ ] RabbitMQ Management UI доступен на `http://localhost:15672` (логин `smp`, пароль `smp`)
- [ ] Web Dashboard доступен на `http://localhost:3000`
- [ ] `./scripts/smoke-test.sh` завершается `🎉 ALL CHECKS PASSED`

## Что указать в Issue

- Ссылку на ветку
- Результат прогона smoke (скриншот или текст вывода `ALL CHECKS PASSED`)
- Версию Docker Desktop и объём выделенной памяти (если были проблемы)
