# 🚀 РУКОВОДСТВО ПО DOCKER ДЕПЛОЮ

## ✅ ПРОЕКТ ГОТОВ К DOCKER ДЕПЛОЮ

Все критические проблемы исправлены. Проект готов к запуску в Docker.

---

## 🔧 ЧТО БЫЛО ИСПРАВЛЕНО

### 1. ✅ Безопасность
- SECRET_KEY читается из переменной окружения
- DEBUG контролируется через env (по умолчанию False)
- ALLOWED_HOSTS настраивается через env
- .env добавлен в .gitignore

### 2. ✅ База данных
- Поддержка DATABASE_URL (приоритет)
- Fallback на POSTGRES_* переменные
- Health checks для ожидания готовности БД

### 3. ✅ Redis/Celery
- CELERY_BROKER_URL из env
- Автоопределение Redis host из broker URL
- Graceful degradation если Redis недоступен

### 4. ✅ Статика и медиа
- Автоматический collectstatic в entrypoint
- Персистентные volumes для media/static
- Создание директорий в Dockerfile

### 5. ✅ Инициализация
- entrypoint.sh для миграций и setup
- Ожидание готовности БД (netcat)
- Автосоздание нужных директорий

### 6. ✅ Мониторинг
- Flower для Celery (порт 5555)
- Health checks для всех сервисов
- Логирование настроено

---

## 📋 ИНСТРУКЦИЯ ПО ДЕПЛОЮ

### ШАГ 1: Подготовка окружения

#### 1.1 Создайте .env файл:
```bash
cp env.template .env
```

#### 1.2 Отредактируйте .env (ВАЖНО!):
```env
# Сгенерируйте новый SECRET_KEY:
# python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
SECRET_KEY=ваш-уникальный-секретный-ключ-здесь

DEBUG=False

# Добавьте ваш домен
ALLOWED_HOSTS=localhost,127.0.0.1,your-production-domain.com

# PostgreSQL (измените пароль!)
POSTGRES_DB=plagiarism_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=ваш-надёжный-пароль-здесь

# DATABASE_URL будет собран автоматически docker-compose
DATABASE_URL=postgres://postgres:ваш-пароль@db:5432/plagiarism_db

# Redis (оставить как есть для Docker)
CELERY_BROKER_URL=redis://redis:6379/0
```

---

### ШАГ 2: Сборка и запуск

```bash
# Сборка образов
docker compose build

# Запуск всех сервисов
docker compose up -d

# Проверка статуса
docker compose ps
```

Должны быть running:
- `postgres_db` (PostgreSQL + pgvector)
- `redis`
- `django_web` (Django/Gunicorn)
- `celery_worker` (обработка документов)
- `flower_monitor` (мониторинг)

---

### ШАГ 3: Инициализация

#### 3.1 Создать суперпользователя:
```bash
docker compose exec web python manage.py createsuperuser
```

#### 3.2 (Опционально) Загрузить начальные данные:
```bash
docker compose exec web python manage.py shell -c "from documents.models import Status, Type; Status.objects.get_or_create(id=1, defaults={'name':'На проверке','html_clase':'queue'}); Type.objects.get_or_create(name='курсач')"
```

---

### ШАГ 4: Проверка работоспособности

#### 4.1 Откройте в браузере:
- **Приложение:** http://localhost:8080
- **Admin:** http://localhost:8080/admin/
- **Flower:** http://localhost:5555

#### 4.2 Тест загрузки документа:
1. Войдите в кабинет
2. Загрузите PDF файл
3. Проверьте в Flower что задача выполняется
4. Документ должен обработаться за 10-30 сек

#### 4.3 Проверьте логи:
```bash
# Логи приложения
docker compose logs -f web

# Логи Celery
docker compose logs -f celery

# Логи БД
docker compose logs -f db
```

---

## 🐛 РЕШЕНИЕ ПРОБЛЕМ

### Проблема: Контейнеры не стартуют
```bash
# Проверьте логи
docker compose logs

# Пересоберите с нуля
docker compose down -v
docker compose build --no-cache
docker compose up -d
```

### Проблема: БД недоступна
```bash
# Проверьте что БД запущена
docker compose ps db

# Проверьте логи
docker compose logs db

# Перезапустите
docker compose restart db web
```

### Проблема: Celery не обрабатывает задачи
```bash
# Проверьте Redis
docker compose exec redis redis-cli ping

# Проверьте логи worker
docker compose logs celery

# Перезапустите
docker compose restart celery
```

### Проблема: Статика не загружается
```bash
# Пересоберите статику в контейнере
docker compose exec web python manage.py collectstatic --clear --noinput

# Проверьте volume
docker volume ls | grep static
```

### Проблема: Media файлы пропадают
```bash
# Проверьте volume
docker volume ls | grep media

# Проверьте права
docker compose exec web ls -la /app/Folder/media/
```

---

## 📊 МОНИТОРИНГ ПРОДАКШЕНА

### Flower Dashboard:
```
http://your-server:5555
```
Показывает:
- Активные задачи
- Success/Failure rate
- Время выполнения
- Количество воркеров

### Логи в реальном времени:
```bash
docker compose logs -f --tail=100
```

### Использование ресурсов:
```bash
docker stats
```

---

## 🔒 БЕЗОПАСНОСТЬ ДЛЯ ПРОДАКШЕНА

### 1. Измените пароли:
- Сгенерируйте новый SECRET_KEY
- Установите сильный пароль БД
- Не используйте дефолтные креды

### 2. Настройте HTTPS:
```yaml
# Добавьте nginx в docker-compose.yml
nginx:
  image: nginx:alpine
  ports:
    - "80:80"
    - "443:443"
  volumes:
    - ./nginx.conf:/etc/nginx/nginx.conf
    - ./ssl:/etc/nginx/ssl
```

### 3. Ограничьте доступ:
- Flower требует аутентификацию
- Закройте порты БД/Redis снаружи
- Используйте firewall

### 4. Backup:
```bash
# Backup БД
docker compose exec db pg_dump -U postgres plagiarism_db > backup.sql

# Backup volumes
docker run --rm -v ad_media_files:/data -v $(pwd):/backup alpine tar czf /backup/media_backup.tar.gz /data
```

---

## 🎯 CHECKLIST ПЕРЕД ДЕПЛОЕМ

- [ ] .env файл создан и заполнен
- [ ] SECRET_KEY уникальный
- [ ] DEBUG=False
- [ ] ALLOWED_HOSTS содержит ваш домен
- [ ] Пароли БД изменены с дефолтных
- [ ] docker compose build завершился успешно
- [ ] docker compose up -d запустил все сервисы
- [ ] Миграции применены
- [ ] Суперпользователь создан
- [ ] Тестовая загрузка документа работает
- [ ] Flower показывает задачи
- [ ] Логи без критических ошибок

---

## 📈 МАСШТАБИРОВАНИЕ

### Увеличение workers:
```yaml
celery:
  deploy:
    replicas: 3  # 3 worker процесса
```

### Горизонтальное масштабирование web:
```bash
docker compose up -d --scale web=3
```

Добавьте nginx как load balancer.

---

## 🎉 ИТОГ

Проект **ПОЛНОСТЬЮ ГОТОВ** к Docker деплою с:
- ✅ Безопасными настройками через env
- ✅ Автоматическими миграциями
- ✅ Персистентными данными
- ✅ Асинхронной обработкой
- ✅ Мониторингом через Flower
- ✅ Health checks всех сервисов

Следуйте инструкциям выше для деплоя!
