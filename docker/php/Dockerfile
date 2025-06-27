# 1. Builder: копируем весь проект и устанавливаем зависимости, если есть composer.json
FROM composer:2 AS builder
WORKDIR /app
COPY . .
RUN if [ -f composer.json ]; then \
      composer install --no-dev --optimize-autoloader; \
    fi

# 2. Runtime: FrankenPHP + Caddy
FROM dunglas/frankenphp:1-php8.4.7-bookworm
WORKDIR /app

# Копируем код и vendor из билдера
COPY --from=builder /app /app

# Caddyfile
COPY docker/php/Caddyfile /etc/caddy/Caddyfile

# Создаём и правим владельца папки для хранения pki
RUN mkdir -p /data/caddy/pki \
 && chown -R 1000:1000 /data/caddy

# Непривилегированный пользователь
RUN addgroup --gid 1000 appgroup \
 && useradd --uid 1000 --gid appgroup --home-dir /app --no-create-home --shell /usr/sbin/nologin appuser \
 && chown -R appuser:appgroup /app
USER appuser

ENTRYPOINT ["frankenphp", "run", "--config", "/etc/caddy/Caddyfile"]
