# Docker deployment tutorial for Nuxt

Этот документ объясняет, как мы задеплоили Nuxt-проект на сервер через Docker и системный NGINX.
Цель не только повторить команды, а понять логику процесса: какие технологии участвуют, зачем они нужны,
что означает каждое действие и какие есть альтернативы.

## 1. Общая картина

У нас есть Nuxt-приложение. В режиме разработки оно запускается командой:

```bash
npm run dev
```

Но на сервере production-приложение обычно не запускают так же, как локально. Для production Nuxt сначала
собирается:

```bash
npm run build
```

После сборки Nuxt создает папку:

```text
.output
```

Внутри нее лежит готовый production-сервер:

```text
.output/server/index.mjs
```

Именно его можно запускать командой:

```bash
node .output/server/index.mjs
```

Docker нужен для того, чтобы не настраивать Node.js, зависимости и окружение вручную прямо на сервере.
Мы описываем окружение один раз в `Dockerfile`, а затем Docker сам собирает и запускает приложение одинаково
на любой машине.

Итоговая схема такая:

```text
Internet
  |
  v
system NGINX :80/:443
  |
  v
http://127.0.0.1:3030
  |
  v
Docker container with Nuxt app
```

Снаружи пользователь открывает:

```text
https://tasks.gromitsoft.ru
```

А внутри сервера NGINX проксирует запросы в Docker-контейнер:

```text
http://127.0.0.1:3030
```

## 2. Какие технологии мы проверяли

Перед деплоем на сервере важно понять, какие инструменты уже установлены.

Мы проверяли:

```bash
cat /etc/os-release
git --version
docker --version
docker compose version
nginx -v
```

### `cat /etc/os-release`

Показывает операционную систему сервера.

Это важно, потому что команды установки зависят от дистрибутива. Например, на Ubuntu и Debian обычно
используется `apt`, на CentOS/Rocky/AlmaLinux - `dnf` или `yum`.

### `git --version`

Git нужен, чтобы клонировать репозиторий проекта на сервер:

```bash
git clone <repo-url>
```

А потом обновлять проект:

```bash
git pull
```

### `docker --version`

Docker Engine нужен, чтобы собирать и запускать контейнеры.

Контейнер - это изолированная среда, внутри которой есть все, что нужно приложению: Node.js, production-сборка,
зависимости и команда запуска.

### `docker compose version`

Docker Compose нужен, чтобы запускать проект декларативно через файл:

```text
docker-compose.yml
```

Без Compose можно было бы запускать контейнер длинной командой `docker run`, но Compose удобнее:
все настройки лежат в одном файле, их проще читать, хранить в git и повторять.

### `nginx -v`

NGINX нужен как reverse proxy.

Он принимает публичные HTTP/HTTPS-запросы от пользователей и перенаправляет их во внутреннее приложение.
Также NGINX обычно отвечает за домены и TLS-сертификаты.

## 3. Почему Node.js на сервер ставить не обязательно

Обычно Nuxt требует Node.js. Но при Docker-деплое Node.js нужен не на самом сервере, а внутри контейнера.

В нашем `Dockerfile` используется образ:

```dockerfile
FROM node:22-alpine
```

Это означает: Docker берет готовую Linux-среду, где уже установлен Node.js 22.

Поэтому на хост-сервере нам нужны:

```text
git
docker
docker compose
nginx
```

А Node.js на хосте не обязателен.

Альтернатива: деплоить без Docker. Тогда на сервере пришлось бы ставить Node.js нужной версии,
запускать `npm ci`, `npm run build`, держать процесс через `pm2` или `systemd`. Это рабочий вариант,
но он сильнее привязан к конкретному серверу.

## 4. Что такое Dockerfile

`Dockerfile` - это рецепт сборки образа.

Образ - это шаблон будущего контейнера. Контейнер - это запущенный экземпляр образа.

В проекте добавлен файл:

```text
Dockerfile
```

Его логика:

```dockerfile
FROM node:22-alpine AS deps
WORKDIR /app

COPY package*.json ./
RUN npm ci
```

На первом этапе Docker ставит зависимости.

Почему копируются только `package*.json`, а не весь проект сразу?
Потому что Docker умеет кешировать слои. Если исходный код изменился, но `package.json` и
`package-lock.json` не изменились, зависимости можно не устанавливать заново.

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build
```

На втором этапе Docker копирует зависимости и код проекта, затем собирает Nuxt production build.

```dockerfile
FROM node:22-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV HOST=0.0.0.0
ENV PORT=3030

COPY --from=build /app/.output ./.output

EXPOSE 3030

CMD ["node", ".output/server/index.mjs"]
```

На третьем этапе создается финальный контейнер. В него попадает только готовая папка `.output`.
Это называется multi-stage build.

Плюсы multi-stage build:

- финальный образ меньше;
- в production-контейнере нет исходников, которые не нужны для запуска;
- dev-зависимости и промежуточные файлы не тащатся в runtime;
- процесс сборки и запуска разделен.

`HOST=0.0.0.0` означает, что приложение слушает не только `localhost` внутри контейнера, а все сетевые
интерфейсы контейнера. Это нужно, чтобы Docker мог пробросить порт наружу.

`PORT=3030` означает, что Nuxt будет слушать порт `3030`.

## 5. Что такое `.dockerignore`

Файл `.dockerignore` говорит Docker, какие файлы не надо отправлять в build context.

Build context - это набор файлов, который Docker получает для сборки образа.

Мы исключаем:

```text
node_modules
.nuxt
.output
.env
.git
```

Почему:

- `node_modules` должны устанавливаться внутри контейнера, а не копироваться с локальной машины;
- `.nuxt` и `.output` являются результатами локальной сборки;
- `.env` содержит секреты и не должен попадать в Docker image;
- `.git` не нужен для запуска приложения.

Это ускоряет сборку и уменьшает риск случайно упаковать секреты.

## 6. Что такое `docker-compose.yml`

`docker-compose.yml` описывает, как запускать контейнер.

В нашем проекте:

```yaml
services:
  app:
    build:
      context: .
    container_name: aspro-gromit
    restart: unless-stopped
    env_file:
      - .env
    environment:
      NODE_ENV: production
      HOST: 0.0.0.0
      PORT: 3030
    ports:
      - "127.0.0.1:3030:3030"
```

### `build.context: .`

Говорит Docker Compose собирать образ из текущей папки, где лежит `Dockerfile`.

### `container_name`

Задает понятное имя контейнера:

```text
aspro-gromit
```

Так его легче искать в списке:

```bash
docker ps
```

### `restart: unless-stopped`

Говорит Docker автоматически перезапускать контейнер, если он упал или если сервер перезагрузился.

Исключение: если мы сами остановили контейнер командой:

```bash
docker compose down
```

### `env_file: .env`

Передает переменные окружения из файла `.env` внутрь контейнера.

Это важно, потому что адрес API, подключение к базе и секреты не должны быть зашиты в код.

### `ports: "127.0.0.1:3030:3030"`

Это одна из самых важных строк.

Она означает:

```text
host 127.0.0.1:3030 -> container 3030
```

То есть приложение доступно только с самого сервера:

```bash
curl http://127.0.0.1:3030
```

Но снаружи напрямую по адресу:

```text
http://45.8.96.18:3030
```

оно не должно быть доступно.

Это сделано специально, потому что публичный вход должен идти через NGINX.

Альтернатива:

```yaml
ports:
  - "3030:3030"
```

Так порт открылся бы наружу на всех интерфейсах сервера. Это проще для быстрой проверки, но хуже для production:
появляется лишняя публичная точка входа.

## 7. Что такое `.env` и можно ли брать локальный файл

`.env` - это файл с переменными окружения.

Его можно взять из локального проекта как основу, но нельзя копировать бездумно. Нужно проверить, подходят ли
значения именно для сервера.

Пример:

```env
GROMIT_API_BASE_URL="https://gromit-knolege.duckdns.org"
GROMIT_AUTH_EMAIL="pm@grom-it.ru"
GROMIT_API_TIMEOUT_MS=30000

DATABASE_URL="postgres://user:password@host:5432/database"

AUTH_SESSION_SECRET="long-random-secret"

NUXT_PUBLIC_APP_URL="https://tasks.gromitsoft.ru"
NUXT_PUBLIC_GROMIT_AUTH_EMAIL="pm@grom-it.ru"
```

### `GROMIT_API_BASE_URL`

Адрес внешнего API старой системы, куда Nuxt-приложение ходит серверным прокси.

### `DATABASE_URL`

Строка подключения к PostgreSQL.

Важный момент: если приложение запущено в Docker, то `localhost` внутри `DATABASE_URL` означает сам контейнер,
а не сервер.

Если база находится на том же сервере, часто нельзя просто написать:

```env
DATABASE_URL="postgres://user:password@localhost:5432/database"
```

Потому что внутри контейнера `localhost` - это контейнер.

Варианты:

- указать IP сервера или адрес базы, доступный из контейнера;
- подключить контейнер к сети, где доступен PostgreSQL;
- запускать PostgreSQL отдельным сервисом в том же `docker-compose.yml`;
- использовать специальный host gateway, если это осознанно настроено.

### `AUTH_SESSION_SECRET`

Секрет для подписи сессий.

Его нельзя хранить в git. Генерируется так:

```bash
openssl rand -base64 48
```

### `NUXT_PUBLIC_APP_URL`

Публичный адрес приложения:

```env
NUXT_PUBLIC_APP_URL="https://tasks.gromitsoft.ru"
```

Переменные с префиксом `NUXT_PUBLIC_` доступны клиентской части приложения, поэтому туда нельзя класть секреты.

## 8. Зачем нужен системный NGINX

Docker-контейнер запускает приложение, но не обязан заниматься доменом и HTTPS.

NGINX решает задачи:

- принимает запросы на `80` и `443`;
- понимает домены через `server_name`;
- проксирует запросы во внутренний порт приложения;
- обслуживает TLS-сертификат;
- может держать несколько сайтов на одном сервере и IP.

Например, на одном сервере могут жить:

```text
gromitsoft.ru        -> основной сайт
tasks.gromitsoft.ru  -> Nuxt-приложение задач
```

Оба домена указывают на один IP, а NGINX решает, куда отправить запрос, по заголовку `Host`.

## 9. Что такое DNS и поддомен

Домен:

```text
gromitsoft.ru
```

Поддомен:

```text
tasks.gromitsoft.ru
```

Чтобы поддомен начал указывать на сервер, в DNS-зоне `gromitsoft.ru` нужна A-запись:

```text
Host: tasks
Type: A
Value: 45.8.96.18
```

Это означает:

```text
tasks.gromitsoft.ru -> 45.8.96.18
```

Если основной сайт уже живет на этом же сервере, это нормально. NGINX различает сайты по `server_name`.

## 10. NGINX-конфиг

Мы создали отдельный конфиг:

```text
/etc/nginx/sites-available/tasks.gromitsoft.ru
```

Пример HTTP-конфига:

```nginx
server {
    listen 80;
    server_name tasks.gromitsoft.ru;

    location / {
        proxy_pass http://127.0.0.1:3030;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### `listen 80`

NGINX принимает обычный HTTP-трафик.

После установки сертификата Certbot добавляет HTTPS-блоки или изменяет конфиг под `443`.

### `server_name tasks.gromitsoft.ru`

Указывает, для какого домена работает этот блок.

Именно этой строки не хватало, когда Certbot сказал:

```text
Could not automatically find a matching server block
```

Certbot выпустил сертификат, но не нашел NGINX-блок, куда его установить.

### `proxy_pass http://127.0.0.1:3030`

Говорит NGINX перенаправлять запросы в Nuxt-контейнер.

### `proxy_set_header`

Передает приложению информацию об исходном запросе:

- какой домен запросил пользователь;
- какой реальный IP у пользователя;
- какой протокол был снаружи: HTTP или HTTPS.

Без этих заголовков приложение или backend могут видеть не настоящего пользователя, а только NGINX.

## 11. Certbot и HTTPS

Certbot получает TLS-сертификат Let's Encrypt и настраивает NGINX.

Команда:

```bash
certbot --nginx -d tasks.gromitsoft.ru
```

Что происходит:

1. Certbot проверяет, что домен `tasks.gromitsoft.ru` указывает на этот сервер.
2. Выпускает сертификат Let's Encrypt.
3. Пытается найти NGINX-блок с `server_name tasks.gromitsoft.ru`.
4. Если находит - сам добавляет HTTPS-настройки.
5. Если не находит - сохраняет сертификат, но просит установить его позже.

У нас была ситуация:

```text
The certificate was saved, but could not be installed
Could not automatically find a matching server block
```

Это означало: сертификат уже выпущен, но NGINX-конфиг еще не был готов.

Решение:

1. Создать NGINX server block с `server_name tasks.gromitsoft.ru`.
2. Проверить NGINX:

```bash
nginx -t
```

3. Перезагрузить NGINX:

```bash
systemctl reload nginx
```

4. Установить уже выпущенный сертификат:

```bash
certbot install --cert-name tasks.gromitsoft.ru
```

Certbot также создает автоматическое продление сертификата. Поэтому вручную каждые 90 дней обновлять
сертификат обычно не нужно.

## 12. Команды запуска проекта

Первый запуск:

```bash
cd /opt
git clone <repo-url> aspro-gromit
cd /opt/aspro-gromit
cp .env.example .env
nano .env
docker compose up -d --build
```

Проверка контейнера:

```bash
docker compose ps
```

Логи:

```bash
docker compose logs -f app
```

Проверка приложения изнутри сервера:

```bash
curl http://127.0.0.1:3030/api/health
```

Проверка публичного сайта:

```bash
curl -I https://tasks.gromitsoft.ru
```

## 13. Как обновлять проект

Когда в репозитории появились новые изменения:

```bash
cd /opt/aspro-gromit
git pull
docker compose up -d --build
```

Что делает эта команда:

- пересобирает Docker image, если код изменился;
- запускает новый контейнер;
- применяет настройки из `docker-compose.yml`;
- оставляет сервис в фоне.

После обновления можно очистить старые неиспользуемые образы:

```bash
docker image prune -f
```

## 14. Как останавливать и перезапускать

Остановить:

```bash
docker compose down
```

Запустить:

```bash
docker compose up -d
```

Перезапустить без пересборки:

```bash
docker compose restart app
```

Посмотреть контейнеры:

```bash
docker ps
```

Посмотреть все контейнеры, включая остановленные:

```bash
docker ps -a
```

## 15. Healthcheck

В `docker-compose.yml` добавлен healthcheck:

```yaml
healthcheck:
  test: ["CMD", "node", "-e", "fetch('http://127.0.0.1:3030/api/health').then((r) => process.exit(r.ok ? 0 : 1)).catch(() => process.exit(1))"]
  interval: 30s
  timeout: 5s
  retries: 3
  start_period: 20s
```

Он периодически проверяет:

```text
http://127.0.0.1:3030/api/health
```

Если endpoint отвечает успешно, контейнер считается healthy.

Это удобно для диагностики:

```bash
docker compose ps
```

## 16. Частые ошибки

### Docker daemon не запущен

Ошибка может выглядеть так:

```text
failed to connect to the docker API
```

Решение:

```bash
systemctl status docker
systemctl start docker
systemctl enable docker
```

### Certbot не нашел server block

Ошибка:

```text
Could not automatically find a matching server block
```

Причина: в NGINX нет блока с нужным `server_name`.

Решение: создать конфиг NGINX для домена, проверить `nginx -t`, затем выполнить:

```bash
certbot install --cert-name tasks.gromitsoft.ru
```

### Приложение работает локально, но сайт не открывается

Проверить по порядку:

```bash
curl http://127.0.0.1:3030/api/health
nginx -t
systemctl status nginx
curl -I https://tasks.gromitsoft.ru
```

Если первый `curl` работает, а публичный сайт нет, проблема чаще всего в NGINX, DNS или сертификате.

Если первый `curl` не работает, проблема в Docker-контейнере или приложении.

### База данных недоступна

Если в `DATABASE_URL` указан `localhost`, а приложение запущено в Docker, это может быть причиной.

Нужно помнить:

```text
localhost внутри контейнера != localhost сервера
```

## 17. Альтернативные варианты деплоя

### Без Docker: Node.js + PM2

Можно поставить Node.js на сервер, собрать проект и запустить через PM2:

```bash
npm ci
npm run build
pm2 start .output/server/index.mjs --name aspro-gromit
```

Плюсы:

- проще понять новичку;
- меньше Docker-слоя.

Минусы:

- нужно следить за версией Node.js на сервере;
- сложнее повторить окружение;
- зависимости живут прямо на хосте.

### Docker + NGINX в одном Compose

Можно запускать и приложение, и NGINX в Docker Compose.

Плюсы:

- весь стек описан в одном месте;
- удобно для новых серверов.

Минусы:

- если на сервере уже есть системный NGINX и другие сайты, проще использовать существующий NGINX;
- нужно аккуратно делить порты `80` и `443`.

### Docker + системный NGINX

Это наш вариант.

Плюсы:

- приложение изолировано в контейнере;
- системный NGINX управляет доменами и сертификатами;
- удобно держать несколько сайтов на одном сервере;
- приложение не открывает свой порт наружу.

Минусы:

- нужно понимать две части: Docker и NGINX;
- конфиги лежат в разных местах: проект в `/opt/aspro-gromit`, NGINX в `/etc/nginx`.

## 18. Как объяснить это на собеседовании

Короткая версия:

> Я упаковал Nuxt-приложение в Docker через multi-stage build. На этапе build выполняется `npm ci` и
> `npm run build`, а в production-слой копируется только `.output`. Контейнер запускает
> `.output/server/index.mjs` на порту `3030`. Через Docker Compose приложение публикуется только на
> `127.0.0.1:3030`, поэтому оно не доступно напрямую из интернета. Публичный доступ идет через системный
> NGINX, который по `server_name tasks.gromitsoft.ru` проксирует запросы в контейнер. HTTPS настроен через
> Certbot и Let's Encrypt.

Развернутая версия:

> Я разделяю responsibilities: Docker отвечает за runtime приложения, зависимости и повторяемую сборку,
> а NGINX отвечает за публичный web entrypoint, домен, HTTPS и reverse proxy. Это позволяет не ставить Node.js
> на хост и не смешивать приложение с системным окружением сервера. Переменные окружения передаются через
> `.env`, который не хранится в git. Для проверки использую `docker compose ps`, `docker compose logs`,
> локальный health endpoint и `nginx -t`.

## 19. Главная мысль

Docker не заменяет сервер и не заменяет NGINX.

Docker отвечает на вопрос:

```text
Как надежно собрать и запустить приложение?
```

NGINX отвечает на вопрос:

```text
Как пользователю попасть в приложение по домену и HTTPS?
```

DNS отвечает на вопрос:

```text
На какой сервер указывает доменное имя?
```

Когда эти три роли разделены, деплой становится понятным:

```text
DNS -> NGINX -> Docker -> Nuxt
```
