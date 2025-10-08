# 🚀 Быстрый старт

## 1. Создайте .env файл (вручную):
```
POSTGRES_DB=plagiarism_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
```

## 2. Запустите систему:
```bash
# Сборка и запуск
docker compose up -d --build

# Проверка статуса
docker compose ps

# Просмотр логов
docker compose logs -f celery
```

## 3. Откройте в браузере:
- **Приложение:** http://localhost:8080/documents/cabinet/
- **Flower (мониторинг):** http://localhost:5555

## 4. Тестирование:
1. Загрузите PDF документ через форму
2. Наблюдайте статус: ⏳ В очереди → 🔄 Обрабатывается → ✅ Проверен
3. Проверьте результат в Flower

## 5. Остановка:
```bash
docker compose down
```

## 📝 Важные команды:

### Применить миграции в контейнере:
```bash
docker compose exec web python manage.py migrate
```

### Пересчитать все документы:
```bash
docker compose exec web python manage.py reprocess_documents --all
```

### Создать суперпользователя:
```bash
docker compose exec web python manage.py createsuperuser
```

### Собрать статику:
```bash
docker compose exec web python manage.py collectstatic --noinput
```

---

## 🎯 Что изменилось:
- ✅ Загрузка документов не блокирует интерфейс
- ✅ Обработка происходит в фоне через Celery
- ✅ Продвинутый детектор с множественными метриками
- ✅ Кэширование векторов в Redis для ускорения
- ✅ Прогресс-бары и live-обновления
- ✅ Flower мониторинг очередей

Подробная документация в `ASYNC_SETUP.md`
