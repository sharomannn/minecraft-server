# Minecraft Server (Forge)

Сервер Minecraft на базе [itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server). Forge 1.20.1 — поддержка модов.

## Требования к серверу

- SSH-доступ (публичный ключ в `~/.ssh/authorized_keys`)
- Docker и Docker Compose установлены

## CI/CD (GitHub Actions)

При первом деплое на сервер автоматически создаётся директория `/opt/minecraft-server`, копируются `compose.yaml` и `.env.example`, при отсутствии файла создаётся `rcon_password` из секрета, затем запускается контейнер. Клонировать репозиторий и создавать файлы вручную не нужно.

Деплой запускается при push в `main` или вручную через **Actions → Deploy to Minecraft Server → Run workflow**.

### Секреты GitHub

В **Settings → Secrets and variables → Actions** добавьте:

| Secret           | Значение                              |
|------------------|----------------------------------------|
| `SSH_PRIVATE_KEY`| Приватный ключ для доступа по SSH      |
| `SSH_HOST`       | IP или hostname сервера               |
| `SSH_PORT`       | Порт SSH (например `8989`)            |
| `SSH_USER`       | Пользователь с доступом к Docker      |
| `RCON_PASSWORD`  | Пароль RCON (при первом деплое создаётся файл `rcon_password`) |

### Подключение клиента

Укажите в лаунчере адрес сервера и порт `25565` (например, `46.191.172.248:25565`).

## Запуск вручную

Если нужно поднять сервер без CI/CD: склонируйте репо, создайте файл `rcon_password` с паролем, выполните `docker compose up -d` в директории с `compose.yaml`.

## Моды

Forge поддерживает моды в формате `.jar`. После первого запуска положите файлы в `./data/mods/` на сервере и перезапустите контейнер.
