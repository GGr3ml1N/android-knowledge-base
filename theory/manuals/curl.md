`curl` — клиент для передачи данных по URL (HTTP, HTTPS, FTP и др.)

## СИНОПСИС

curl [ОПЦИИ]... [URL]...

## ОПИСАНИЕ

`curl` выполняет запросы к серверу и выводит ответ в стандартный вывод. Основной инструмент для ручной проверки REST API из терминала. Построен на библиотеке `libcurl`.

По умолчанию выполняется HTTP GET. Многие опции изменяют поведение неявно (см. ЗАМЕЧАНИЯ).

## ОПЦИИ

### Метод и тело запроса

| Опция | Действие |
|---|---|
| `-X, --request <метод>` | Явно задать HTTP-метод (GET, POST, PUT, DELETE...) |
| `-d, --data <данные>` | Тело запроса. Неявно переключает метод на POST. Данные: сырая строка или `ключ=значение` |
| `-d @файл` | Тело запроса прочитать из файла (например, `--data @workout.json`) |
| `--json <данные>` | Сокращение (curl ≥ 7.82): тело + `Content-Type: application/json` + `Accept: application/json`. Эквивалентно `-H ... -d ...` |
| `-H, --header <заголовок>` | Добавить или переопределить заголовок. Формат: `"Имя: значение"` |
| `-G, --get` | Передать данные из `-d` как query-параметры в URL (для GET с параметрами) |

### Вывод ответа

| Опция | Действие |
|---|---|
| `-i, --include` | Вывести заголовки ответа вместе с телом |
| `-I, --head` | Выполнить HEAD: только заголовки, без тела |
| `-v, --verbose` | Подробный режим: исходящие заголовки, TLS-рукопожатие, ответ. Основной инструмент диагностики |
| `-s, --silent` | Не выводить прогресс-бар и ошибки подключения |
| `-o <файл>` | Записать тело ответа в файл |
| `-w, --write-out <формат>` | После тела вывести метрики. Переменные: `%{http_code}`, `%{time_total}`, `%{content_type}`, `%{size_download}` |

### Прочее

| Опция | Действие |
|---|---|
| `-L, --location` | Следовать редиректам (3xx) |
| `-u, --user <логин:пароль>` | HTTP Basic Authentication |
| `-k, --insecure` | Не проверять TLS-сертификат. Только для отладки |
| `--connect-timeout <сек>` | Таймаут на установку соединения |
| `--max-time <сек>` | Общий таймаут запроса |
| `-x, --proxy <host:port>` | Отправить запрос через прокси |

## ПРИМЕРЫ

```bash
# GET: список ресурсов
curl http://localhost:8080/workouts

# GET: одна запись
curl http://localhost:8080/workouts/1

# GET: с заголовками ответа (статус виден сразу)
curl -i http://localhost:8080/workouts/99

# POST: создать запись (современный короткий вариант)
curl -X POST --json '{"name":"Силовая","date":1730000000000}' http://localhost:8080/workouts

# POST: классический вариант с явным заголовком
curl -X POST -H "Content-Type: application/json" \
     -d '{"name":"Силовая","date":1730000000000}' \
     http://localhost:8080/workouts

# POST: тело из файла
curl -X POST -H "Content-Type: application/json" \
     --data @workout.json \
     http://localhost:8080/workouts

# Только код статуса (удобно для скриптов)
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/workouts

# Диагностика: что реально ушло на сервер
curl -v http://localhost:8080/workouts

# GET с query-параметром
curl -G -d "page=2" http://localhost:8080/workouts
# эквивалентно: curl "http://localhost:8080/workouts?page=2"
```

## ЗАМЕЧАНИЯ

- `-d` без `-X` переключает метод на POST.
- `-d` по умолчанию выставляет `Content-Type: application/x-www-form-urlencoded`. REST API на Spring ждёт JSON → без `-H "Content-Type: application/json"` (или `--json`) вернётся **415 Unsupported Media Type**.
- Заголовок `-H "Content-Type: ..."` переопределяет умолчание от `-d`.
- Прогресс-бар (`#` в выводе) не ошибка — уберите через `-s` или `-o`.

## СМ. ТАКЖЕ

docker(1), psql(1), документация проекта (theory/spring)
