# Конфигурация

## Переменные окружения

| Переменная | Используется в | Назначение и поведение |
|---|---|---|
| PORT | app.js | Порт HTTP-сервера; по умолчанию 3002 |
| BLACKLISTMODE | previewGenerator.js | Читается и отражается в логах/ответе пропуска. Не передаётся в менеджер правил и фактически не включает/выключает блокировку |
| WHITELISTMODE | previewGenerator.js | При true разрешает парсинг только доменов из parsingRules |
| IS_POSTERS_ON_BY_DEFAULT | previewGenerator.js, helpers.js | Разрешает постеры для доменов без отдельного правила; явный posterAllowed=false имеет приоритет |
| DEBUG_LEVEL | previewGenerator.js, sitesCrawler.js, logger.js, debugHandler.js | Управляет режимами подробности логирования; значение verbose также включает полную загрузку страницы crawler |
| LOG_LEVEL | logger.js | Порог логирования целевого логгера |
| LOGLEVEL | logger.js | Альтернативное имя переменной уровня логирования |
| YOUTUBE_API_KEY | fetchYoutubeMetadata.js | Ключ YouTube Data API v3 |
| KINOPOISK_API_KEY | kinopoiskPreviewHandler.js | Ключ Kinopoisk Unofficial API |

Для `utils/logger.js` поддерживаются уровни trace, verbose, debug, info, warn, error, fatal. Сравнение значения с таблицей уровней выполняется без приведения регистра, поэтому используйте имена в нижнем регистре.

## Конфигурационные файлы

### parsingRules.json

Массив правил для доменов. Используемые поля:

- `domain` — домен или базовый домен;
- `classification` — текстовая категория (safe, moderate, other, warn);
- `notes` — пояснение;
- `fastmode` — стратегия ранней остановки crawler;
- `onlyPoster` — вернуть пустые title и description для обычного предпросмотра;
- `posterAllowed` — явное разрешение/запрет постера.

Правило применяется как к указанному домену, так и к его поддоменам. В текущем файле 105 записей; есть повтор записи для mail.ru. Значение classification сейчас не включает самостоятельную модерацию контента.

### blockedDomains.json

Массив доменов, которые менеджер правил пропускает без обычного парсинга. Текущее содержимое:
- hdrezka.ag
- rezka.ag
- youtube.com

Проверка выполняется независимо от `BLACKLISTMODE`.

### Файлы env

В репозитории есть `debugWhitelist.env`; package.json также ссылается на `verbose.env` и `debug.env`, которых в текущем дереве нет.

В `debugWhitelist.env` заданы имена переменных для YouTube API, уровня отладки и Kinopoisk API. Значения секретов документация намеренно не воспроизводит.

**Безопасность:** на момент аудита `debugWhitelist.env` отслеживался Git и содержал непустые API-ключи. Следует считать их раскрытыми, перевыпустить и удалить из отслеживаемой конфигурации. Удаление файла из последующих коммитов само по себе не удалит его значения из истории Git.

## Запуск с env-файлами

В package.json определены команды:

- `npm run start_verbose` — ожидает `verbose.env`;
- `npm run start_debug` — ожидает `debug.env`;
- `npm run start_debug_whitelist` — читает `debugWhitelist.env`.

Не запускайте последний скрипт до проверки и ротации содержащихся в файле секретов. Для локального запуска предпочтительно передавать актуальные значения через переменные окружения или защищённый локальный env-файл, который не добавляется в Git.

## CORS

Разрешённые origins и методы сейчас заданы непосредственно в `app.js`, а не через переменные окружения.
