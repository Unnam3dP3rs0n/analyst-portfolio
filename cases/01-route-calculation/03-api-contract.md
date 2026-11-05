# Контракт

## Эндпоинт и авторизация

`GET https://api.routing.yandex.net/v2/route`

Авторизация: `apikey` 

## Параметры запроса

| Параметр | Тип | Обязательный | Пример | Описание |
|---|---|---|---|---|
| `apikey` | string | Да | `<token>` | Ключ доступа из Кабинета Разработчика |
| `waypoints` | string | Да | `55.7558,37.6173\|55.7520,37.6155\|55.7600,37.6200` | Координаты точек в формате `lat1,lon1\|lat2,lon2\|...` (WGS84). Для `driving` и `truck` до 50 точек |
| `mode` | string | Нет | `truck` | Тип транспорта: `driving` (по умолчанию) или `truck` |
| `departure_time` | integer | Нет | `1763182800` | UNIX-время отправления для учёта пробок. Не может быть в прошлом |
| `traffic` | string | Нет | `disabled` | `disabled`  без учёта пробок |
| `avoid_tolls` | boolean | Нет | `true` | `true` объезд платных дорог (только для `driving`/`truck`) |

**Дополнительные параметры для `mode=truck`:**

| Параметр | Тип | Пример | Описание |
|---|---|---|---|
| `weight` | float | `10` | Масса ТС, тонны |
| `axle_weight` | float | `6` | Нагрузка на ось, тонны |
| `max_weight` | float | `20` | Разрешённая максимальная масса, тонны |
| `height` | float | `3.5` | Высота, метры |
| `width` | float | `2.5` | Ширина, метры |
| `length` | float | `8` | Длина, метры |
| `payload` | float | `15` | Максимальная грузоподъёмность, тонны |
| `eco_class` | integer | `4` | Экологический класс |
| `has_trailer` | boolean | `true` | Наличие прицепа. Учитывает знак 3.7 «Движение с прицепом запрещено» |

## Формат ответа

    {
      "traffic_type": "forecast",
      "route": {
        "legs": [
          {
            "status": "OK",
            "steps": [
              {
                "length": 1116.16,
                "duration": 206.48,
                "mode": "truck"
              },
              {
                "length": 6356.56,
                "duration": 1143.63,
                "mode": "truck"
              }
            ]
          }
        ],
        "flags": {
          "hasTolls": false,
          "hasNonTransactionalTolls": false
        }
      }
    }

| Поле | Тип | Пример | Описание |
|---|---|---|---|
| `traffic_type` | string | `forecast` | Тип информации о пробках: `realtime` (на момент запроса), `forecast` (прогноз), `disabled` (без пробок) |
| `route.legs[].status` | string | `OK` | Статус построения маршрута: `OK` или `FAIL` |
| `route.legs[].steps[].length` | float | `1116.16` | Длина шага, метры |
| `route.legs[].steps[].duration` | float | `206.48` | Время шага, секунды |
| `route.legs[].steps[].mode` | string | `truck` | Режим маршрутизации |
| `route.flags.hasTolls` | boolean | `false` | Маршрут проходит через платные дороги |

## Ошибки

Ошибки возвращаются в поле `errors`.

| Код | Причина | Пример тела ответа |
|---|---|---|
| `400` | Запрос не содержит обязательных параметров | `{ "errors": ["parameter 'waypoints' is missing"] }` |
| `401` | Нет `apikey` или неверный ключ | `{ "errors": ["Key not found"] }` |
| `429` | Превышено количество запросов | `{ "errors": ["Counter total limit exceeded. Limit: XXX, current value: YYY"] }` |
| `500` / `504` | Системная ошибка сервера | `{ "errors": ["..."] }` |
