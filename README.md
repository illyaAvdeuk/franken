# Franken Symfony Docker Stack

## Описание
Готовый к работе Docker‑стек для Symfony‑приложения на FrankenPHP + Caddy (HTTP-only) + Postgres + RabbitMQ + Redis.

## Структура
- `docker-compose.yml` — все сервисы и порты (HTTP на 8087).
- `.env` — переменные окружения.
- `docker/php/`:
  - `Dockerfile` — multi‑stage билд: Composer → FrankenPHP.
  - `Caddyfile` — Caddy в режиме HTTP-only (`auto_https off`), ловит все хосты на порту 80 и проксирует в FrankenPHP.

## Быстрый старт

1. Убедитесь, что в корне `~/Study/franken` лежит ваш Symfony‑проект. Иначе используйте `public/index.php` для заглушки.
2. Скопируйте и отредактируйте `.env`.

   ```bash
   cp .env.example .env
   # В .env:
   APP_HTTP_PORT=8087
   ```

3. Запустите стек:

   ```bash
   docker-compose up -d --build
   ```

4. Проверьте доступность приложения по HTTP:

   ```bash
   # на localhost
   curl http://localhost:8087/

   # с подстановкой Host=franken.local
   curl -H "Host: franken.local" http://localhost:8087/
   ```

5. Выполните Symfony‑команды внутри контейнера `php`:

   ```bash
   docker-compose exec php bin/console doctrine:migrations:migrate
   docker-compose exec php bin/console cache:clear
   ```

6. Для перезапуска:

   ```bash
   docker-compose down
   docker-compose up -d --build
   ```

## Примечания

- **Порт:** HTTP производится на контейнерном 80, проброшен на 8087 хоста (`APP_HTTP_PORT`).
- **Caddy:** работает в чистом HTTP‑режиме (`auto_https off`), ловит все запросы на порт 80 внутри контейнера и проксирует на FrankenPHP.
- **Пользователь:** в контейнере используется `UID:GID 1000:1000` (appuser/appgroup).
- **Symfony:** запускается в режиме `dev` (из `.env`: `SYMFONY_ENV`).

## Файловая структура

```
~/Study/franken/
├── .env
├── docker-compose.yml
├── README.md
└── docker/
    └── php/
        ├── Dockerfile
        └── Caddyfile
```

В этой конфигурации стек готов к работе по HTTP на порту 8087. При подключении реального Symfony‑проекта все запросы (в том числе фронт‑роуты) будут попадать в `public/index.php`. Если потребуется вернуть HTTPS — достаточно добавить в `Caddyfile` секцию `tls` и проброс портов 8443.
