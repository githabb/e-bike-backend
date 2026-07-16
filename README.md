# E-Bike Rental — деплой на Render через Docker

## Що важливо знати перед деплоєм

1. **`.env` не входить в архів.** Скопіюй `.env.example` у `.env` для локальної розробки
   і заповни своїми реальними ключами. На Render ці змінні задаються окремо
   (див. нижче) — файл `.env` в образ не потрапить (див. `.dockerignore`).

2. **`database.sqlite` теж не входить в архів.** Контейнери на Render — ефемерні:
   при кожному деплої/рестарті файлова система пересоздається, і локальний файл БД
   зникне. Щоб дані не втрачалися, потрібен **Persistent Disk** (див. нижче).

## Локальний запуск (без Docker)

```bash
npm install
cp .env.example .env   # і впиши туди свої ключі
npm run dev             # або: npm start
```

## Локальний запуск через Docker

```bash
docker build -t e-bike-rental .
docker run -p 3000:3000 --env-file .env -v $(pwd)/data:/usr/src/app/data e-bike-rental
```

## Деплой на Render

1. Залий цей проєкт у Git-репозиторій (GitHub/GitLab) — `.gitignore` вже
   налаштований так, щоб `.env` і `database.sqlite` туди не потрапили.
2. У Render створи **Web Service** з цього репозиторію.
3. У **Settings → Build & Deploy → Environment** обери **Docker**
   (замість Node) — Dockerfile лежить у корені репозиторію, шлях вказувати не потрібно.
4. Поля **Build Command** / **Start Command** у Docker-режимі не використовуються —
   усе вже описано в `Dockerfile` (`CMD ["node", "server.js"]`).
5. У **Environment → Environment Variables** додай вручну всі змінні
   зі свого `.env` (PORT можна не вказувати — Render підставить свій,
   сервер вже читає `process.env.PORT`).
6. У **Settings → Disks** додай **Persistent Disk**, змонтуй, наприклад,
   у `/usr/src/app/data`, і вкажи шлях до БД через змінну `DATABASE`
   (наприклад `DATABASE=/usr/src/app/data/database.sqlite`), щоб файл
   переживав редеплої.
7. Збережи і запусти деплой — Render збере образ на базі `node:20-bookworm`,
   де помилка `GLIBC_2.38 not found` при встановленні `sqlite3` вже не виникає.

## Структура проєкту

- `server.js` — Express-бекенд
- `script.js`, `style.css`, `*.html` — фронтенд
- `Dockerfile` — збірка образу для Render
- `.env.example` — шаблон змінних середовища (без реальних ключів)
