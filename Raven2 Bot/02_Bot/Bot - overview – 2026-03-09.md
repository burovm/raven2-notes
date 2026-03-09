# Raven 2 – состояние проекта на 2026-03-09

## 1. Краткое описание
- Цель: бот-помощник по рынку Raven 2, который собирает все мои сделки (buy/sell) и строит аналитику.
- Архитектура: Linux backend (bot-debian, FastAPI + SQLite) + Windows VM с OCR-клиентом + Obsidian как «мозг проекта».

## 2. Что уже сделано
- Proxmox:
  - Поднят rocketsrv, создана VM `bot-debian`, доступ по SSH под пользователем `bmv`.
- Backend на bot-debian:
  - Репозиторий `raven2-bot` в `~/projects/raven2-bot`.
  - FastAPI + SQLAlchemy + SQLite: модели `terramonds`, `snapshots`, `lots`.
  - Endpoint `POST /snapshots` принимает JSON c `taken_at`, `source`, `lots[...]` и сохраняет в БД.
  - Собран Docker-образ backend’а (`python:3.11-slim`, FastAPI, uvicorn, SQLAlchemy, Pydantic).
  - docker-compose.yml поднимает сервис `raven2-backend` на порту 8000.
  - Тестовый `curl`/Postman POST создаёт snapshot и пишет в SQLite (проверка прошла).
- Obsidian:
  - Vault `C2 Bot` со структурой:
    - `00Inbox`
    - `01Infrastructure` (Proxmox, VM `bot-debian`)
    - `02Bot` (`Bot - overview`, `BotDB - blogdb schema`)
    - `03WindowsVMRaven2`
    - `90Journal`
  - Obsidian-вольт подключён к Git (`raven2-notes`), есть рабочий цикл commit/push.

- Windows VM (Raven 2):
  - Создана Win11 VM в Proxmox.
  - Поставлены VirtIO-драйверы (virtio-win-gt-x64), сеть работает, Win11 пингуется до `bot-debian`.
  - Включён RDP, настроены базовые параметры (сон/экран).
  - Установлен Python 3.14.3 + pip, поставлен пакет `requests`.

## 3. Что работает «от конца до конца»
- Backend:
  - `docker compose up -d` поднимает `raven2-backend`.
  - `http://<bot-debian>:8000/docs` открывается, POST `/snapshots` пишет данные в SQLite.
- Сеть:
  - Win11 VM видит `bot-debian` по IP, `ping` успешен.

## 4. Что запланировано (следующие шаги)
1. Windows-клиент:
   - Написать простой Python-скрипт на Win11, который шлёт тестовый JSON на `/snapshots` (без OCR).
   - Отладить формат payload так, чтобы backend спокойно принимал данные из Windows.
2. OCR и реальные сделки:
   - Выбрать OCR-стек (варианты: Tesseract, сервисы, что угодно локальное).
   - Определить формат JSON из OCR: `terramond_code`, `terramond_name`, `unit_price`, `quantity`, `direction`, `taken_at`, `source`.
   - Сделать первую версию скрипта: «сделать скрин → OCR → собрать lots[] → POST /snapshots».
3. Аналитика:
   - Добавить эндпоинты вида:
     - `GET /stats/terramond/{code}` (avg price, суммарный объём).
     - `GET /stats/terramond/{code}/by_weekday_hour` (heatmap активности).
4. Документация:
   - Расширить `02Bot/BotDB - blogdb schema` схемой таблиц и примером JSON.
   - Документировать простой E2E сценарий: «сделка в игре → скрин → запись в БД → просмотр статистики».

## 5. Риски и открытые вопросы
- Выбор OCR-инструмента (качество распознавания и стабильность).
- Удобство запуска клиента на Win11 (в фоне / по горячей клавише / по расписанию).
- Надо ли докеризовать Windows-клиента или достаточно «голого» Python-скрипта.

## 6. Ближайший фокус (1–2 дня)
- Завершить тестовый HTTP-клиент на Win11 и добиться стабильного POST в backend.
- Описать формат JSON для реальных лотов и зафиксировать его в Obsidian.
- Начать прототипировать OCR-пайплайн под одну-две типовые формы сделки.
