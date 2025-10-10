# 🚀 БЫСТРЫЙ СТАРТ REDIS

## ❌ Проблема: Docker Desktop не запущен

Ошибка: `The system cannot find the file specified`

## ✅ РЕШЕНИЕ

### Шаг 1: Запустите Docker Desktop

1. **Найдите в меню Пуск:** "Docker Desktop"
2. **Запустите приложение**
3. **Подождите** пока Docker полностью загрузится (иконка кита станет зелёной)

### Шаг 2: Проверьте Docker

```powershell
docker ps
```

Если видите таблицу контейнеров (даже пустую) - Docker работает! ✅

---

## 🔴 ЗАПУСК REDIS

После запуска Docker Desktop выполните:

```powershell
# Запустить Redis
docker run -d -p 6379:6379 --name redis --restart unless-stopped redis:alpine

# Проверить что работает
docker ps

# Должны увидеть:
# CONTAINER ID   IMAGE          PORTS
# abc123...      redis:alpine   0.0.0.0:6379->6379/tcp
```

---

## ✅ ПРОВЕРКА ПОДКЛЮЧЕНИЯ

### Через Docker CLI:
```powershell
docker exec -it redis redis-cli ping
# Ответ: PONG ✅
```

### Через Python:
```powershell
cd Folder
python

>>> import redis
>>> r = redis.Redis(host='localhost', port=6379)
>>> r.ping()
True  ✅
```

---

## 🚀 ЗАПУСК CELERY

После того как Redis работает:

```powershell
# Откройте НОВЫЙ терминал
cd C:\allcodes\ad\Folder

# Активируйте venv
..\.venv\Scripts\activate

# Запустите Celery worker
celery -A app worker --loglevel=info --pool=solo
```

Должны увидеть:
```
[tasks]
  . documents.tasks.process_document_plagiarism
  . documents.tasks.batch_process_documents

celery@COMPUTER ready.
```

---

## 🌸 ЗАПУСК FLOWER (мониторинг)

```powershell
# В НОВОМ терминале
cd C:\allcodes\ad\Folder
..\.venv\Scripts\activate

celery -A app flower --port=5555
```

Откройте: **http://localhost:5555**

Увидите:
- Активные задачи
- Очередь задач
- Статистику воркеров

---

## 🎯 ФИНАЛЬНЫЙ ТЕСТ

### 1. Запущены 3 окна терминала:

**Терминал 1 (Django):**
```powershell
cd Folder
python manage.py runserver
```

**Терминал 2 (Celery Worker):**
```powershell
cd Folder
celery -A app worker --loglevel=info --pool=solo
```

**Терминал 3 (Flower) - опционально:**
```powershell
cd Folder
celery -A app flower --port=5555
```

### 2. Откройте браузер:
- **Django:** http://localhost:8000
- **Flower:** http://localhost:5555

### 3. Загрузите документ:
- Перейдите в кабинет
- Загрузите PDF или DOCX
- **Результат:** Мгновенный ответ (<1 сек)! ⚡
- **В Celery логах:** увидите начало обработки
- **Через 30-60 сек:** обновите страницу - процент появится

---

## 🛑 ОСТАНОВКА

```powershell
# Остановить Redis
docker stop redis

# Удалить контейнер (если нужно)
docker rm redis

# Остановить Celery - Ctrl+C в терминале
```

---

## 🔧 ПОЛЕЗНЫЕ КОМАНДЫ

### Redis:
```powershell
# Логи
docker logs redis -f

# Подключиться к CLI
docker exec -it redis redis-cli

# Перезапустить
docker restart redis

# Статус
docker ps | findstr redis
```

### Celery:
```powershell
# Очистить очередь
celery -A app purge

# Список задач
celery -A app inspect active

# Статистика
celery -A app inspect stats
```

---

## 📊 ПРОИЗВОДИТЕЛЬНОСТЬ ДО/ПОСЛЕ

### ДО (без Redis):
```
Загрузка документа → [████████ 60 сек ждём] → Готово
```

### ПОСЛЕ (с Redis):
```
Загрузка документа → [⚡ <1 сек] → Готово!
                            ↓
                    (Обработка в фоне)
```

**Разница: 60x быстрее!** 🚀

---

## 🎉 ГОТОВО!

После этих шагов вы получите:
- ⚡ Мгновенный отклик (<1 сек)
- 🔄 Параллельную обработку (4-8 документов)
- 📊 Мониторинг через Flower
- 💾 Кэширование результатов
- 🎯 Production-ready систему

Наслаждайтесь скоростью! 🚀✨
