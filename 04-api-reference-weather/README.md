# API прогноза погоды Open-Meteo: метод `GET /v1/forecast`

Справочник для разработчиков: как получить текущую погоду и почасовой прогноз для точки по координатам.

> Документ написан для портфолио по открытому API [Open-Meteo](https://open-meteo.com/). Примеры запросов и ответов проверены на реальном API в ноябре 2026 года. Полный перечень параметров — в [официальной документации](https://open-meteo.com/en/docs).

---

## Содержание

- [Быстрый старт](#быстрый-старт)
- [Общие сведения](#общие-сведения)
- [Запрос](#запрос)
- [Ответ](#ответ)
- [Ошибки](#ошибки)
- [Примеры](#примеры)
- [Ограничения](#ограничения)

---

## Быстрый старт

Получите текущую температуру в Санкт-Петербурге одним запросом:

```bash
curl "https://api.open-meteo.com/v1/forecast?latitude=59.94&longitude=30.31&current=temperature_2m"
```

Ответ содержит температуру в поле `current.temperature_2m` (фрагмент ответа, время по Гринвичу):

```json
{
  "current": {
    "time": "2026-11-20T12:00",
    "interval": 900,
    "temperature_2m": 3.8
  }
}
```

---

## Общие сведения

| Параметр | Значение |
|---|---|
| Базовый URL | `https://api.open-meteo.com` |
| Протокол | HTTPS |
| Формат ответа | JSON, кодировка UTF-8 |
| Аутентификация | Не требуется для некоммерческого использования |
| Формат времени | ISO 8601 без секунд, например `2026-11-20T15:00` |

---

## Запрос

```
GET /v1/forecast
```

### Параметры запроса

| Параметр | Тип | Обяз. | Описание | Пример |
|---|---|---|---|---|
| `latitude` | число | Да | Широта точки в градусах, от −90 до 90 | `59.94` |
| `longitude` | число | Да | Долгота точки в градусах, от −180 до 180 | `30.31` |
| `current` | строка | Нет | Переменные текущей погоды через запятую | `temperature_2m,wind_speed_10m` |
| `hourly` | строка | Нет | Переменные почасового прогноза через запятую | `temperature_2m` |
| `forecast_days` | целое | Нет | Количество дней прогноза. По умолчанию — 7 | `1` |
| `timezone` | строка | Нет | Часовой пояс в формате базы IANA. По умолчанию — `GMT` | `Europe/Moscow` |

> **Важно.** Значение `timezone` содержит символ `/`. В URL его нужно кодировать как `%2F`: `Europe%2FMoscow`. Большинство HTTP-клиентов делают это автоматически.

### Часто используемые переменные

| Переменная | Описание | Единица по умолчанию |
|---|---|---|
| `temperature_2m` | Температура воздуха на высоте 2 м | °C |
| `wind_speed_10m` | Скорость ветра на высоте 10 м | км/ч |
| `relative_humidity_2m` | Относительная влажность на высоте 2 м | % |
| `precipitation` | Сумма осадков | мм |

---

## Ответ

### Код ответа

`200 OK` — запрос выполнен.

### Пример ответа

Запрос:

```bash
curl "https://api.open-meteo.com/v1/forecast?latitude=59.94&longitude=30.31&current=temperature_2m,wind_speed_10m&hourly=temperature_2m&forecast_days=1&timezone=Europe%2FMoscow"
```

Ответ (массивы `hourly` сокращены):

```json
{
  "latitude": 59.940144,
  "longitude": 30.303894,
  "generationtime_ms": 0.0587,
  "utc_offset_seconds": 10800,
  "timezone": "Europe/Moscow",
  "timezone_abbreviation": "GMT+3",
  "elevation": 0.0,
  "current_units": {
    "time": "iso8601",
    "interval": "seconds",
    "temperature_2m": "°C",
    "wind_speed_10m": "km/h"
  },
  "current": {
    "time": "2026-11-20T15:00",
    "interval": 900,
    "temperature_2m": 3.8,
    "wind_speed_10m": 13.0
  },
  "hourly_units": {
    "time": "iso8601",
    "temperature_2m": "°C"
  },
  "hourly": {
    "time": ["2026-11-20T00:00", "2026-11-20T01:00", "..."],
    "temperature_2m": [2.0, 2.0, "..."]
  }
}
```

### Поля ответа

| Поле | Тип | Описание |
|---|---|---|
| `latitude`, `longitude` | число | Координаты ближайшего узла расчётной сетки. Могут немного отличаться от запрошенных |
| `generationtime_ms` | число | Время формирования ответа на сервере, мс |
| `utc_offset_seconds` | целое | Смещение часового пояса относительно UTC, с |
| `timezone` | строка | Часовой пояс, в котором указано время в ответе |
| `timezone_abbreviation` | строка | Сокращённое обозначение часового пояса |
| `elevation` | число | Высота точки над уровнем моря, м |
| `current_units` | объект | Единицы измерения для полей объекта `current` |
| `current` | объект | Текущие значения запрошенных переменных |
| `current.time` | строка | Время, на которое актуальны значения |
| `current.interval` | целое | Длительность интервала, за который усреднены значения, с |
| `hourly_units` | объект | Единицы измерения для полей объекта `hourly` |
| `hourly.time` | массив строк | Метки времени прогноза с шагом 1 час |
| `hourly.<переменная>` | массив чисел | Значения переменной. Элемент с индексом `i` соответствует `hourly.time[i]` |

---

## Ошибки

При ошибке API возвращает код `400 Bad Request` и объект:

```json
{
  "error": true,
  "reason": "Latitude must be in range of -90 to 90°. Given: 999.0."
}
```

| Поле | Тип | Описание |
|---|---|---|
| `error` | логическое | Всегда `true` при ошибке |
| `reason` | строка | Причина ошибки на английском языке |

Частые причины:

| Причина | Начало текста `reason` | Как исправить |
|---|---|---|
| Координата вне допустимого диапазона | `Latitude must be in range of -90 to 90°` | Проверьте, что `latitude` от −90 до 90, `longitude` от −180 до 180 |
| Неизвестное имя переменной в `current` или `hourly` | `Invalid value: Cannot initialize …` | Сверьте имя с [перечнем переменных](https://open-meteo.com/en/docs) |
| Некорректный часовой пояс | `Invalid timezone` | Используйте имя из базы IANA, например `Europe/Moscow` |

---

## Примеры

### Python

Пример использует библиотеку `requests`. Установите её, если она ещё не установлена:

```bash
pip install requests
```

```python
import requests

params = {
    "latitude": 59.94,
    "longitude": 30.31,
    "current": "temperature_2m,wind_speed_10m",
    "timezone": "Europe/Moscow",
}

response = requests.get("https://api.open-meteo.com/v1/forecast", params=params, timeout=10)
data = response.json()

if response.status_code != 200:
    raise RuntimeError(data["reason"])

print(f"Температура: {data['current']['temperature_2m']} °C")
```

### JavaScript

Пример работает в браузере и в Node.js 18 и новее (как модуль `.mjs`, где разрешён `await` на верхнем уровне).

```javascript
const url = new URL("https://api.open-meteo.com/v1/forecast");
url.search = new URLSearchParams({
  latitude: 59.94,
  longitude: 30.31,
  current: "temperature_2m",
  timezone: "Europe/Moscow",
});

const response = await fetch(url);
const data = await response.json();

if (!response.ok) throw new Error(data.reason);
console.log(`Температура: ${data.current.temperature_2m} °C`);
```

---

## Ограничения

- Бесплатное использование разрешено только в некоммерческих целях. Для коммерческих проектов нужен платный тариф.
- Количество запросов в сутки ограничено. Актуальные лимиты — на странице [условий использования](https://open-meteo.com/en/terms).
- Не опрашивайте API чаще, чем раз в 15 минут для одной точки: текущие значения обновляются с интервалом `current.interval` (900 с).
