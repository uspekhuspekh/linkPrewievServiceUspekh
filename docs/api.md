# HTTP API

## GET /api/parse

Получает URL в query-параметре, формирует предпросмотр и возвращает JSON.

### Запрос

```http
GET /api/parse?url=https%3A%2F%2Fexample.com
```

Пример с curl:

```bash
curl --get "http://localhost:3002/api/parse" \
  --data-urlencode "url=https://example.com"
```

### Параметры

| Параметр | Обязательный | Описание |
|---|---|---|
| url | Да | URL страницы для предпросмотра |

Текущая реализация проверяет только наличие непустого значения. Отдельной валидации схемы, hostname, допустимого порта или публичности IP-адреса в HTTP-маршруте нет. Вопросы безопасности описаны в [аудите](audit.md).

### Обычный успешный ответ

```json
{
  "url": "https://example.com",
  "title": "Example Domain",
  "description": "Example description",
  "image": "https://example.com/preview.png"
}
```

Это иллюстративный пример формы результата, а не гарантированный ответ конкретного сайта. Значения title, description и image зависят от доступного HTML и метаданных.

### Специализированные ответы

YouTube handler может вернуть дополнительные поля:

```json
{
  "url": "https://www.youtube.com/watch?v=VIDEO_ID",
  "title": "Video title",
  "description": "Video description",
  "image": "https://i.ytimg.com/...",
  "channel": "Channel name",
  "duration": "PT4M13S",
  "views": "1000"
}
```

Kinopoisk handler формирует объект с `url`, `title`, `description`, `image`.

При `onlyPoster` title и description обычного предпросмотра будут пустыми строками.

### Дополнительные поля кеша

В зависимости от пути запроса ответ может получить:
- `_fromCache: true` — найден готовый кешированный результат;
- `_shared: true` — запрос получил результат уже выполнявшейся обработки того же URL.

Эти поля добавляются middleware, а не ядром парсинга.

### Пропуск обработки

Если правила запрещают или пропускают URL, возвращается объект примерно следующей формы:

```json
{
  "message": "Skip parsing because of rules",
  "rules": {
    "skipParsing": true
  },
  "whitelistMode": false,
  "blacklistMode": false
}
```

Полный объект rules содержит дополнительные поля. Конкретная форма зависит от причины пропуска.

Для URL с распознанным файловым расширением возможен ответ:

```json
{
  "url": "https://example.com/document.pdf",
  "note": "Skipped because file link"
}
```

### Ошибки и HTTP-коды

| HTTP-код | Условие в текущем коде | Ответ |
|---|---|---|
| 200 | Handler вернул результат, включая некоторые неуспешные результаты | JSON с возвращённым значением |
| 400 | Не передан параметр url | `{"error":"No URL provided"}` |
| 500 | Handler выбросил неперехваченное исключение | `{"error":"Parser error"}` |
| 500 | Promise общего запроса завершился ошибкой | `{"error":"Parser error (shared)"}` |

Важно: не каждая ошибка парсинга превращается в HTTP 500. Например, YouTube и Kinopoisk handlers при многих ошибках возвращают `null`, а внешний catch в `parseUrl()` возвращает строку `"test"`. Middleware может отправить такие значения как ответ с HTTP 200.

### CORS

В `app.js` заданы разрешённые origins:
- `https://tv.2ch.su`
- `https://citx.space`

Разрешённые методы CORS: GET, OPTIONS, POST. Фактически в текущем API реализован только GET-маршрут `/api/parse`.

CORS — браузерный механизм, а не аутентификация API и не замена серверной валидации входящих URL.

## Прочие HTTP-маршруты

| Метод | Путь | Назначение |
|---|---|---|
| GET | / | Возвращает тестовую страницу `views/index.html` |

Отдельные URL для статических ресурсов обслуживаются Express static middleware из каталогов `public` и `views`.
