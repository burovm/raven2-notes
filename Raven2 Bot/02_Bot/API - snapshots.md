# API: `POST /snapshots` — контракт JSON (Linux backend `raven2-bot`)

## Назначение

`POST /snapshots` — эндпоинт backend’а `raven2-bot`, который принимает один «снимок аукциона» с лотами террамондов и сохраняет его в SQLite (`snapshots`, `terramonds`, `lots`).

URL (на bot-debian):

- `http://192.168.50.185:8000/snapshots`
    

Метод:

- `POST`
    

Content-Type:

- `application/json`
    

---

## Структура запроса

Корневой объект JSON:

json

`{   "taken_at": "2026-03-08T10:00:00Z",  "source": "win_vm1_client1",  "lots": [ ... ] }`

Поля:

- `taken_at` (string, обязательное)
    
    - Время снятия аукциона в формате ISO 8601, **обязательно** с указанием часового пояса.
        
    - Рекомендуемый формат: `YYYY-MM-DDTHH:MM:SSZ` (UTC) или `YYYY-MM-DDTHH:MM:SS+03:00` и т.п.
        
    - Backend хранит это поле как `datetime` (в SQLite) и использует для агрегирования по дню недели/часу.
        
- `source` (string или null, необязательное)
    
    - Идентификатор источника снимка: какая VM/машина/клиент сделал этот снимок.
        
    - Примеры: `"manual_test"`, `"win_vm1_client1"`, `"main_pc_client2"`.
        
    - Можно использовать для фильтрации и отладки (если будет несколько клиентов).
        
- `lots` (array, обязательное)
    
    - Массив объектов, каждый объект описывает один лот террамонда в этом снимке.
        
    - Если при парсинге скрина аукциона ничего не нашли — можно отправить пустой массив `[]`, но само поле `lots` должно присутствовать.
        

Элемент массива `lots` (один лот):

json

`{   "terramond_code": "wood_basic",  "terramond_name": "Basic Wood",  "unit_price": 123.45,  "quantity": 100,  "direction": "sell" }`

Поля лота:

- `terramond_code` (string, обязательное)
    
    - Внутренний код террамонда (slug/идентификатор), который Windows‑парсер должен **стабильно** присваивать одному и тому же террамонду.
        
    - Используется как уникальное поле в таблице `terramonds.code`.
        
    - Примеры: `"wood_basic"`, `"terra_rare_fire"`, `"ore_iron_low"`.
        
- `terramond_name` (string, обязательное)
    
    - Человеческое имя террамонда, как оно видно в клиенте Raven 2.
        
    - Используется для удобства и может меняться, но `terramond_code` должен оставаться стабильным.
        
    - Примеры: `"Basic Wood"`, `"Rare Fire Terra"`.
        
- `unit_price` (number, обязательное)
    
    - Цена за единицу террамонда в этом лоте.
        
    - Тип: float (в JSON — число).
        
    - В БД хранится как `FLOAT`.
        
- `quantity` (number, обязательное)
    
    - Количество единиц террамонда в лоте (например, 100 штук).
        
    - Тип: int (в JSON — целое число).
        
- `direction` (string, обязательное)
    
    - Направление сделки: `"buy"` или `"sell"`.
        
    - Как парсер определяет это — задача Windows‑слоя (по UI аукциона).
        
    - В БД это строка, по которой можно фильтровать.
        

---

## Поведение backend’а

- При каждом вызове `POST /snapshots` создаётся новая запись в таблице `snapshots` (`taken_at`, `source`).
    
- Для каждого элемента `lots[]`:
    
    - backend ищет или создаёт запись в `terramonds` по `terramond_code`;
        
    - создаёт запись в `lots` с `snapshot_id`, `terramond_id`, `unit_price`, `quantity`, `direction`.
        

Ответ:

- Успех (HTTP 200):
    

json

`{ "snapshot_id": 1 }`

- При ошибке валидации Pydantic (неправильные типы/отсутствующие поля) FastAPI вернёт 422 с описанием ошибок.
    

---

## Примеры запросов

**1. Минимальный пример с одним лотом**

json

`{   "taken_at": "2026-03-08T10:00:00Z",  "source": "manual_test",  "lots": [    {      "terramond_code": "wood_basic",      "terramond_name": "Basic Wood",      "unit_price": 123.45,      "quantity": 100,      "direction": "sell"    }  ] }`

**2. Пример с несколькими лотами разных террамондов (один снимок)**

json

`{   "taken_at": "2026-03-08T12:30:00Z",  "source": "win_vm1_client1",  "lots": [    {      "terramond_code": "wood_basic",      "terramond_name": "Basic Wood",      "unit_price": 110.0,      "quantity": 50,      "direction": "sell"    },    {      "terramond_code": "wood_basic",      "terramond_name": "Basic Wood",      "unit_price": 115.0,      "quantity": 150,      "direction": "sell"    },    {      "terramond_code": "terra_rare_fire",      "terramond_name": "Rare Fire Terra",      "unit_price": 999.0,      "quantity": 10,      "direction": "buy"    }  ] }