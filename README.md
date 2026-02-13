# Minecraft Server (Forge)

Сервер Minecraft на базе [itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server). Forge 1.20.1 — поддержка модов.

## Требования к серверу

- SSH-доступ (публичный ключ в `~/.ssh/authorized_keys`)
- Docker и Docker Compose установлены

## CI/CD (GitHub Actions)

При первом деплое на сервер автоматически создаётся директория `~/minecraft-server` (в домашней папке пользователя из `SSH_USER`), копируются `compose.yaml` и `.env.example`, при отсутствии файла создаётся `rcon_password` из секрета, затем запускается контейнер. Клонировать репозиторий и создавать файлы вручную не нужно.

Деплой запускается при push в `main` или вручную через **Actions → Deploy to Minecraft Server → Run workflow**.

### Подготовка к первому деплою

**На своей машине (или в любом месте с `ssh-keygen`):**

1. Сгенерировать ключ для GitHub Actions (без пароля):

   ```bash
   ssh-keygen -t ed25519 -f github_actions_key -N ""
   ```

   Появятся файлы `github_actions_key` (приватный) и `github_actions_key.pub` (публичный).

2. Добавить **публичный** ключ на сервер:
   - Скопировать содержимое `github_actions_key.pub` (одна строка).
   - На сервере под пользователем, под которым будет выполняться деплой (этот же логин — значение `SSH_USER`):
     ```bash
     mkdir -p ~/.ssh
     echo "вставьте_содержимое_github_actions_key.pub" >> ~/.ssh/authorized_keys
     chmod 700 ~/.ssh
     chmod 600 ~/.ssh/authorized_keys
     ```
   - Либо с хоста: `ssh-copy-id -i github_actions_key.pub -p ПОРТ USER@HOST` (если порт не 22, указать `-p`).

3. Проверить подключение перед настройкой GitHub:

   ```bash
   ssh -i github_actions_key -p ПОРТ USER@HOST "echo OK"
   ```

   Должно вывести `OK` без запроса пароля.

**В GitHub (Settings → Secrets and variables → Actions):**

4. Добавить секрет `SSH_PRIVATE_KEY`: скопировать **весь** содержимое файла `github_actions_key`, включая строки `-----BEGIN OPENSSH PRIVATE KEY-----` и `-----END OPENSSH PRIVATE KEY-----`. Не удалять переводы строк и не добавлять лишних пробелов.
5. Добавить остальные секреты: `SSH_HOST`, `SSH_PORT`, `SSH_USER`, `RCON_PASSWORD`.

### Секреты GitHub

В **Settings → Secrets and variables → Actions** добавьте:

| Secret            | Значение                                                                 |
|-------------------|--------------------------------------------------------------------------|
| `SSH_PRIVATE_KEY` | Полное содержимое файла приватного ключа (включая BEGIN/END), без пароля |
| `SSH_HOST`        | IP или hostname сервера                                                 |
| `SSH_PORT`        | Порт SSH (например `8989`)                                              |
| `SSH_USER`        | Пользователь с доступом к Docker                                        |
| `RCON_PASSWORD`   | Пароль RCON (при первом деплое создаётся файл `rcon_password`)           |

### Подключение клиента

Укажите в лаунчере адрес сервера и порт `25565` (например, `46.191.172.248:25565`).

### Устранение неполадок

- **unable to authenticate / no supported methods remain** — сервер не принял ключ. Проверить: (1) публичный ключ добавлен в `~/.ssh/authorized_keys` **того пользователя**, который указан в `SSH_USER`; (2) в секрете `SSH_PRIVATE_KEY` вставлен именно приватный ключ целиком; (3) `SSH_HOST`, `SSH_PORT`, `SSH_USER` соответствуют реальному доступу к серверу.
- **Permission denied** при создании директории — деплой идёт в `~/minecraft-server` (домашняя папка пользователя `SSH_USER`), sudo не нужен. Если в workflow всё ещё указан `/opt/...`, обновите репозиторий до последней версии.
- Убедитесь, что на сервере установлены Docker и Docker Compose (workflow выполняет `docker compose`).

## Запуск вручную

Если нужно поднять сервер без CI/CD: склонируйте репо, создайте файл `rcon_password` с паролем, выполните `docker compose up -d` в директории с `compose.yaml`.

## Моды

Forge поддерживает моды в формате `.jar`. После первого запуска положите файлы в `./data/mods/` на сервере и перезапустите контейнер.
