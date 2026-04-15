# Kittygram

## Описание проекта

Kittygram — это про котиков

### Основные функции:
- Публикация фотографий котиков с описанием
- Выбор цвета и породы котика
- Возможность отмечать понравившиеся фото
- Просмотр всех опубликованных котиков
- Адаптивный дизайн для мобильных устройств

## Стек технологий

### Backend:
- Python 3.9
- Django REST Framework
- PostgreSQL
- Gunicorn
- Ruff (линтер)

### Frontend:
- React
- Node.js 18
- npm

### Инфраструктура:
- Docker
- Docker Compose
- Nginx
- GitHub Actions (CI/CD)
- Docker Hub

## Развертывание проекта

### Предварительные требования

Убедитесь, что на вашем сервере или локальной машине установлены:
- Docker
- Docker Compose
- Git

### Локальный запуск

1. **Клонируйте репозиторий:**
```bash
git clone https://github.com/ваш_username/kittygram_final.git
cd kittygram_final
```

2. **Создайте файл `.env` в корне проекта:**
```bash
touch .env
```

3. **Заполните `.env` файл следующими переменными:**
```env
# Настройки PostgreSQL
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=ваш_пароль_к_бд

# Настройки подключения к БД
DB_HOST=db
DB_PORT=5432

# Настройки Django
SECRET_KEY=ваш_секретный_ключ_django
DEBUG=False
ALLOWED_HOSTS=127.0.0.1,localhost,ваш_домен
```

> **Как сгенерировать SECRET_KEY:**
> Выполните в терминале:
> ```bash
> python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
> ```

4. **Соберите и запустите контейнеры:**
```bash
docker-compose up -d --build
```

5. **Выполните миграции базы данных:**
```bash
docker-compose exec backend python manage.py migrate
```

6. **Соберите статические файлы:**
```bash
docker-compose exec backend python manage.py collectstatic --no-input
```

7. **Создайте суперпользователя (опционально):**
```bash
docker-compose exec backend python manage.py createsuperuser
```

8. **Приложение будет доступно по адресу:**
- http://localhost:9000

### Запуск на удаленном сервере

1. **Выполните шаги 1-3 из локального запуска**

2. **Измените `ALLOWED_HOSTS` в `.env`:**
```env
ALLOWED_HOSTS=ваш_домен.ru,ip_адрес_сервера
```

3. **Обновите `docker-compose.yml` для использования образов с Docker Hub:**
```yaml
services:
  backend:
    image: ваш_username/kittygram_backend:latest
    
  frontend:
    image: ваш_username/kittygram_frontend:latest
    
  gateway:
    image: ваш_username/kittygram_gateway:latest
    ports:
      - "9000:80"
```

4. **Запустите контейнеры:**
```bash
docker-compose pull
docker-compose up -d
docker-compose exec backend python manage.py migrate
docker-compose exec backend python manage.py collectstatic --no-input
```

5. **Настройте Nginx на сервере (пример конфигурации):**
```nginx
server {
    listen 80;
    server_name kittygram.ваш_домен.ru;
    
    location / {
        proxy_pass http://localhost:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

6. **Перезапустите Nginx:**
```bash
sudo nginx -t
sudo systemctl restart nginx
```

## CI/CD

Проект настроен на автоматическую сборку и деплой через GitHub Actions.

### Workflow включает:
1. **Тестирование:**
   - Проверка кода бэкенда с помощью Ruff
   - Запуск тестов бэкенда
   - Запуск тестов фронтенда

2. **Сборка и публикация образов:**
   - Сборка Docker-образов для backend, frontend и gateway
   - Пуш образов в Docker Hub

3. **Уведомления:**
   - Отправка сообщения в Telegram об успешном завершении

### Настройка CI/CD:

Для работы CI/CD необходимо добавить секреты в GitHub:
- `DOCKER_USERNAME` — логин на Docker Hub
- `DOCKER_PASSWORD` — пароль от Docker Hub
- `TELEGRAM_TOKEN` — токен Telegram бота
- `TELEGRAM_TO` — ваш Telegram ID

При пуше в ветку `main` автоматически запускается процесс тестирования и обновления образов.

## Структура проекта

```
kittygram_final/
├── backend/           # Код бэкенда на Django
│   ├── Dockerfile
│   ├── requirements.txt
│   └── ...
├── frontend/          # Код фронтенда на React
│   ├── Dockerfile
│   └── ...
├── nginx/             # Конфигурация Nginx
│   ├── Dockerfile
│   └── nginx.conf
├── docker-compose.yml # Оркестрация контейнеров
├── .env.example       # Пример переменных окружения
└── .github/
    └── workflows/
        └── main.yml   # Конфигурация GitHub Actions
```

## Полезные команды

### Docker Compose

```bash
# Запуск контейнеров
docker-compose up -d

# Остановка контейнеров
docker-compose down

# Просмотр логов
docker-compose logs -f [имя_сервиса]

# Перезапуск конкретного сервиса
docker-compose restart backend

# Обновление образов и перезапуск
docker-compose pull
docker-compose up -d --force-recreate
```

### База данных

```bash
# Создание резервной копии БД
docker-compose exec db pg_dump -U kittygram_user kittygram > backup.sql

# Восстановление из резервной копии
docker-compose exec -T db psql -U kittygram_user kittygram < backup.sql
```
