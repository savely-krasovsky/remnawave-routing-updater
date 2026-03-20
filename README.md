# Remna Routing Updater

Микросервис для автоматического обновления `happRouting` в Remna панели при появлении новых данных в GitHub-репозитории с routing-конфигурацией.

## Как работает

1. При запуске получает текущие настройки подписки из Remna API (`GET /subscription-settings`)
2. Периодически проверяет GitHub-файл с routing-конфигурацией
3. Если источник содержит JSON, собирает из него финальный `happ://routing/onadd/...`
4. При необходимости добавляет дополнительное правило в `DirectSites`
5. Если итоговый deeplink изменился — отправляет обновление в Remna (`PATCH /subscription-settings`)
6. Если изменений нет — ничего не делает

## Быстрый старт

```bash
mkdir remna-routing-updater && cd remna-routing-updater
```

### Внешняя панель (HTTPS)

Создайте файл `.env`:

```env
REMNA_BASE_URL=https://your-host/api
REMNA_TOKEN=your_bearer_token
# EXTRA_DIRECT_SITE=domain:example.com
# GITHUB_RAW_URL=https://raw.githubusercontent.com/hydraponique/roscomvpn-routing/refs/heads/main/HAPP/DEFAULT.JSON
# CHECK_INTERVAL=300
```

Создайте файл `docker-compose.yml`:

```yaml
services:
  routing-updater:
    image: ghcr.io/savely-krasovsky/remnawave-routing-updater:latest
    container_name: remna-routing-updater
    restart: unless-stopped
    env_file:
      - .env
```

### Локальная панель (Docker)

Если RemnaWave панель запущена локально в Docker (образ `remnawave/backend:latest`), контейнер updater нужно подключить к той же сети `remnawave-network` и обращаться к панели по имени контейнера.

Создайте файл `.env`:

```env
REMNA_BASE_URL=http://remnawave-backend:3000/api
REMNA_TOKEN=your_bearer_token
# EXTRA_DIRECT_SITE=domain:example.com
# GITHUB_RAW_URL=https://raw.githubusercontent.com/hydraponique/roscomvpn-routing/refs/heads/main/HAPP/DEFAULT.JSON
# CHECK_INTERVAL=300
```

> `remnawave-backend` — имя контейнера панели, `3000` — порт по умолчанию. Измените при необходимости.

Создайте файл `docker-compose.yml`:

```yaml
services:
  routing-updater:
    image: ghcr.io/savely-krasovsky/remnawave-routing-updater:latest
    container_name: remna-routing-updater
    restart: unless-stopped
    env_file:
      - .env
    networks:
      - remnawave-network

networks:
  remnawave-network:
    name: remnawave-network
    external: true
```

> Сеть `remnawave-network` должна уже существовать (создаётся docker-compose панели RemnaWave).

Запуск:

```bash
docker compose up -d
```

### Сборка из исходников

Если хотите собрать образ самостоятельно:

```bash
git clone https://github.com/lifeindarkside/Remnawave-Routing-update.git
cd Remnawave-Routing-update
cp .env.example .env
# отредактируйте .env
docker build -t remna-routing-updater .
docker compose up -d
```

## Переменные окружения

| Переменная | Обязательная | По умолчанию | Описание |
|---|---|---|---|
| `REMNA_BASE_URL` | да | — | Базовый URL API Remna (например `https://host/api` или `http://remnawave-backend:3000/api`) |
| `REMNA_TOKEN` | да | — | Bearer-токен для авторизации в Remna API |
| `EXTRA_DIRECT_SITE` | нет | — | Дополнительное правило, которое добавляется в `DirectSites` перед обновлением, например `domain:example.com` |
| `GITHUB_RAW_URL` | нет | [DEFAULT.JSON](https://raw.githubusercontent.com/hydraponique/roscomvpn-routing/refs/heads/main/HAPP/DEFAULT.JSON) | URL источника роутинга на GitHub. Поддерживается как JSON, так и готовый deeplink |
| `CHECK_INTERVAL` | нет | `300` | Интервал проверки обновлений (в секундах) |

## Логи

```bash
docker compose logs -f
```

## Лицензия

MIT
