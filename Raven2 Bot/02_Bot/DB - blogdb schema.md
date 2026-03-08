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