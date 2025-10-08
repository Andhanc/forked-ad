# 🖥️ Локальный запуск (без Docker)

## Проблема
Ошибка `ConnectionRefusedError: [WinError 10061]` возникает потому, что Celery пытается подключиться к Redis, который не запущен локально.

## Решения

### ✅ ВАРИАНТ 1: Graceful Degradation (УЖЕ РЕАЛИЗОВАНО)
Система теперь автоматически переключается на синхронную обработку если Redis недоступен.

**Плюсы:**
- Не нужно устанавливать Redis
- Работает сразу

**Минусы:**
- Загрузка документов блокирует браузер на 30-60 сек
- Нет параллелизма
- Нет мониторинга через Flower

**Как использовать:**
Просто запустите как обычно:
```bash
python Folder/manage.py runserver
```
При загрузке документа увидите предупреждение "Redis недоступен, обработка в синхронном режиме..."

---

### ✅ ВАРИАНТ 2: Запустить Redis локально (Windows)

#### Способ A: Redis через Docker (только Redis, без всего проекта)
```bash
docker run -d -p 6379:6379 --name redis redis:7
```

#### Способ B: Redis для Windows (устаревший, не рекомендуется)
1. Скачать: https://github.com/microsoftarchive/redis/releases
2. Установить и запустить как службу Windows

#### Способ C: WSL2 + Redis
```bash
wsl -d Ubuntu
sudo apt update && sudo apt install redis-server
sudo service redis-server start
```

После запуска Redis:
```bash
# Проверка подключения
redis-cli ping
# Должен ответить: PONG

# Запуск Celery worker локально
celery -A Folder.app worker --loglevel=info

# В другом терминале - Django сервер
python Folder/manage.py runserver
```

**Плюсы:**
- Полная асинхронность
- Можно мониторить через Flower
- Параллельная обработка

---

### ✅ ВАРИАНТ 3: Полный Docker (РЕКОМЕНДУЕТСЯ)

#### Шаг 1: Создать `.env` в корне:
```
POSTGRES_DB=plagiarism_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
```

#### Шаг 2: Запустить всё:
```bash
docker compose up -d --build
```

#### Шаг 3: Применить миграции в контейнере:
```bash
docker compose exec web python manage.py migrate
```

#### Шаг 4: Создать суперпользователя (опционально):
```bash
docker compose exec web python manage.py createsuperuser
```

#### Шаг 5: Открыть:
- Приложение: http://localhost:8080
- Flower: http://localhost:5555

**Плюсы:**
- Полная изоляция
- Все сервисы (DB, Redis, Celery) работают
- Продакшен-подобное окружение

---

## 🔧 Текущее состояние системы

### Что работает сейчас (без Redis):
✅ Загрузка документов (медленно, синхронно)  
✅ Продвинутый детектор плагиата  
✅ Все 5+ метрик анализа  
✅ Сохранение detailed_analysis  
❌ Асинхронность (нужен Redis)  
❌ Прогресс-бары в реальном времени  
❌ Flower мониторинг  

### Что работает с Redis:
✅ Всё вышеперечисленное  
✅ Мгновенный ответ формы (<1 сек)  
✅ Параллельная обработка  
✅ Прогресс-бары и live-обновления  
✅ Flower dashboard  
✅ Кэширование векторов  

---

## 🎯 Рекомендация

Для разработки:
1. Запустите только Redis через Docker:
   ```bash
   docker run -d -p 6379:6379 --name redis redis:7
   ```

2. Запустите Celery worker локально:
   ```bash
   celery -A Folder.app worker --loglevel=info --pool=solo
   ```
   Используйте `--pool=solo` на Windows для совместимости.

3. Запустите Django:
   ```bash
   python Folder/manage.py runserver
   ```

4. (Опционально) Flower:
   ```bash
   celery -A Folder.app flower
   ```
   Открыть: http://localhost:5555

Для продакшена/тестирования - используйте полный Docker Compose.

---

## ✨ Итог

Система полностью готова и будет работать в любом режиме:
- **Без Redis:** медленно, но надёжно (синхронно)
- **С Redis локально:** быстро и асинхронно
- **В Docker:** продакшен-готово с полным стеком

Выберите удобный вариант и запускайте! 🚀
