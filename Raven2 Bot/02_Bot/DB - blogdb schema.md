**2026‑03‑08 — текущий статус backend `raven2-bot` (bot-debian, 192.168.50.185:8000)**

Таблицы БД (SQLite `data/raven2.sqlite`):

- `terramonds`: `id`, `code` (unique), `name`.
    
- `snapshots`: `id`, `taken_at` (UTC), `source`.
    
- `lots`: `id`, `snapshot_id` (FK → `snapshots.id`), `terramond_id` (FK → `terramonds.id`), `unit_price`, `quantity`, `direction` (`"buy"` / `"sell"`).
    

Эндпоинты FastAPI:

- `POST /snapshots` — принимает снимок аукциона:
    
    - `taken_at`: datetime (UTC),
        
    - `source`: str | null,
        
    - `lots`: массив объектов `{ terramond_code, terramond_name, unit_price, quantity, direction }`.  
        Сохраняет данные в SQLite и возвращает `{ "snapshot_id": <id> }`.
        
- `GET /stats/terramond/{code}/basic` — базовая статистика по террамонду:
    
    - поля ответа: `terramond_code`, `terramond_name`, `avg_price`, `min_price`, `max_price`, `total_quantity`.
        
- `GET /stats/terramond/{code}/by_weekday_hour` — статистика по дню недели и часу:
    
    - `items[]`: `{ weekday (0=Mon..6=Sun), hour (0..23), avg_price, total_quantity }`.


**Простой сигнал по террамонду (`GET /signals/terramond/{code}/simple`) — версия с фильтрами**

Эндпоинт:

- `GET /signals/terramond/{code}/simple`
    

Параметры:

- `code` — `terramond_code` (например, `wood_basic`).
    
- `current_price` (float, обязательный) — текущая наблюдаемая цена с аукциона.
    
- `direction_filter` (string, query, опциональный, по умолчанию `"sell"`):
    
    - `"sell"` — учитывать только лоты с `direction="sell"`.
        
    - `"buy"` — учитывать только `direction="buy"`.
        
    - `"both"` — учитывать все лоты.
        
- `days` (int, query, опциональный):
    
    - если задано (`days > 0`), средняя цена считается только по снимкам за последние `N` дней от `datetime.utcnow()`.
        
    - если `None`, используется вся доступная история.
        

Ответ (схема `TerramondSimpleSignal`):

json

`{   "terramond_code": "wood_basic",  "terramond_name": "Basic Wood",  "current_price": 90.0,  "avg_price": 123.45,  "deviation_percent": -27.09,  "action": "buy",  "reason": "Текущая цена значительно ниже средней (<= 80% от средней)" }`

Правило:

- Backend вычисляет `avg_price` по фильтрам `direction_filter` и `days`.
    
- `deviation_percent = (current_price - avg_price) / avg_price * 100`.
    
- Решение:
    
    - `current_price <= 0.8 * avg_price` → `action = "buy"`.
        
    - `current_price >= 1.2 * avg_price` → `action = "sell"`.
        
    - иначе → `action = "hold"`.
        
- Если по заданным фильтрам нет данных → `action = "hold"`, `avg_price` и `deviation_percent` = `null`, reason: «Недостаточно исторических данных…».

**Эндпоинт таймлайна цен (`GET /prices/terramond/{code}/timeline`)**

Назначение:

- Вернуть временной ряд цен по одному террамонду для анализа и построения графиков.
    

Параметры:

- `code` — код террамонда (`terramond_code`, например `wood_basic`).
    
- `direction_filter` (query, опционально, по умолчанию `"both"`):
    
    - `"sell"` — только лоты с `direction="sell"`.
        
    - `"buy"` — только `direction="buy"`.
        
    - `"both"` — все лоты.
        
- `days` (query, опционально):
    
    - если `days > 0`, берутся только снимки за последние `N` дней от `datetime.utcnow()`.
        
    - если не задан, используется вся история.
        

Схема ответа (`TerramondPriceTimeline`):

json

`{   "terramond_code": "wood_basic",  "terramond_name": "Basic Wood",  "points": [    {      "taken_at": "2026-03-08T10:00:00",      "unit_price": 123.45,      "quantity": 100,      "direction": "sell"    }  ] }`

Где каждый `points[]` (`TerramondPricePoint`):

- `taken_at` — время снимка аукциона (UTC, как в `snapshots.taken_at`).
    
- `unit_price` — цена за единицу.
    
- `quantity` — количество в лоте.
    
- `direction` — `"buy"` или `"sell"`.