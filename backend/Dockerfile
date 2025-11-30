FROM alpine:3.18 AS builder
WORKDIR /app
COPY . .

FROM php:8.1-cli-alpine

RUN apk add --no-cache bash \
    && docker-php-ext-install pdo pdo_mysql

WORKDIR /var/www/html

COPY --from=builder /app .

RUN chmod +x wait-for-it.sh

EXPOSE 8000
