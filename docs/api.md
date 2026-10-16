# Описание API AccessEvent

## Таблица эндпоинтов

| Метод | URL | Доступ | Вход (JSON) | Ответы |
|---|---|---|---|---|
| POST | /api/register | все | username, email, password, role | 201 user_id; 400 ошибки валидации |
| POST | /api/login | все | email, password | 200 token; 401 неверные данные |
| GET | /api/events | все | — | 200 список мероприятий |
| POST | /api/events | organizer | title, event_date, is_inclusive | 201 event_id; 401 без токена |
| GET | /api/events/<id> | все | — | 200 мероприятие; 404 не найдено |
| PUT | /api/events/<id> | organizer | title, event_date, ... | 200 обновлённое; 403 чужое |
| DELETE | /api/events/<id> | organizer | — | 200 удалено; 403 чужое |
| POST | /api/registrations | user | event_id | 201; 400 повторная; 404 нет события |
| GET | /api/registrations | user | — | 200 список регистраций |
| PUT | /api/registrations/<id> | organizer | status | 200 обновлён; 400 неверный статус |

## Примеры запросов

### POST /api/register — регистрация

Запрос:
```json
{
  "username": "anna",
  "email": "anna@mail.ru",
  "password": "Passw0rd!",
  "role": "user"
}
```

Ответ 201:
```json
{
  "user_id": 1,
  "message": "Пользователь создан"
}
```

Ответ 400 (валидация):
```json
{
  "message": "Пароль должен быть не короче 8 символов"
}
```

### POST /api/login — вход

Запрос:
```json
{
  "email": "anna@mail.ru",
  "password": "Passw0rd!"
}
```

Ответ 200:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "role": "user",
  "user_id": 1
}
```

Ответ 401 (неверные данные):
```json
{
  "message": "Неверный email или пароль"
}
```

### POST /api/events — создание мероприятия (нужен токен организатора)

Заголовок:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json
```

Запрос:
```json
{
  "title": "Тихий кинопросмотр",
  "description": "Инклюзивный показ",
  "event_date": "2026-11-03 19:00:00",
  "location": "Библиотека им. Некрасова",
  "is_inclusive": true,
  "sign_language": true,
  "visual_alerts": true
}
```

Ответ 201:
```json
{
  "message": "Мероприятие создано",
  "id": 5
}
```

Ответ 401 (без токена):
```json
{
  "message": "Токен отсутствует"
}
```

Ответ 403 (не организатор):
```json
{
  "message": "Недостаточно прав"
}
```

### GET /api/events — публичный список

Ответ 200:
```json
[
  {
    "id": 1,
    "title": "Кинопросмотр с сурдопереводом",
    "event_date": "2026-11-01T12:00:00",
    "location": "Библиотека им. Некрасова",
    "organizer_id": 1,
    "accessibility": {
      "is_inclusive": true,
      "sign_language": true,
      "visual_alerts": true
    }
  }
]
```

## Коды ответов

| Код | Значение | Когда |
|---|---|---|
| 200 | OK | Успешный GET/PUT/DELETE |
| 201 | Created | Успешный POST (создание) |
| 400 | Bad Request | Ошибка валидации данных |
| 401 | Unauthorized | Нет токена / неверный токен / неверный логин-пароль |
| 403 | Forbidden | Недостаточно прав (не organizer/admin) |
| 404 | Not Found | Объект не найден |
| 500 | Internal Server Error | Ошибка сервера (баг) |