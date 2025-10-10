# 🔄 АВТОЗАПУСК CELERY

## ❓ Нужно ли запускать Celery вручную?

**НЕТ!** Есть несколько способов автоматизации.

---

## 🎯 ВАРИАНТЫ (от лучшего к худшему)

---

## ✅ ВАРИАНТ 1: Docker Compose (ЛУЧШИЙ)

### Что это?
Запускает **ВСЁ** одной командой:
- PostgreSQL
- Redis
- Django
- **Celery** ← Автоматически!
- Flower (мониторинг)

### Как использовать:

```powershell
# Запустить всё
docker compose up -d

# Проверить статус
docker compose ps

# Остановить всё
docker compose down
```

### Преимущества:
- ✅ Одна команда для всего
- ✅ Автоперезапуск при сбоях
- ✅ Production-ready
- ✅ Изолированное окружение
- ✅ Легко развернуть на сервере

### Когда использовать:
- 🚀 **Production** (рабочий сервер)
- 📦 **Deployment** (развёртывание)
- 🧪 **Полное тестирование**

---

## ✅ ВАРИАНТ 2: BAT-файл + Task Scheduler

### Шаг 1: Создан файл `start_celery.bat`

Уже создан в корне проекта! Просто **двойной клик** для запуска.

### Шаг 2: Добавить в автозагрузку Windows

**Через Task Scheduler:**

1. **Откройте:** `Win+R` → `taskschd.msc`
2. **Создать задачу** → "Create Basic Task"
3. **Имя:** "Celery Worker Plagiarism"
4. **Триггер:** "When I log on" (при входе в систему)
5. **Действие:** "Start a program"
6. **Программа:** `C:\allcodes\ad\start_celery.bat`
7. **Готово!**

Теперь Celery запустится **автоматически** при загрузке Windows.

### Преимущества:
- ✅ Автозапуск при старте системы
- ✅ Работает в фоне
- ✅ Не нужен Docker

### Недостатки:
- ⚠️ Redis нужно запускать отдельно
- ⚠️ Django нужно запускать отдельно
- ⚠️ Менее надёжно чем Docker

---

## ✅ ВАРИАНТ 3: Windows Service (NSSM)

### Что это?
Превращает Celery в **Windows службу** (как антивирус).

### Установка NSSM:

```powershell
# Через Chocolatey
choco install nssm

# Или скачать: https://nssm.cc/download
```

### Настройка:

```powershell
# Создать службу
nssm install CeleryWorker "C:\allcodes\ad\.venv\Scripts\celery.exe"

# Параметры
nssm set CeleryWorker AppParameters "-A app worker --loglevel=info --pool=solo"
nssm set CeleryWorker AppDirectory "C:\allcodes\ad\Folder"

# Запустить
nssm start CeleryWorker
```

### Управление:

```powershell
# Статус
nssm status CeleryWorker

# Остановить
nssm stop CeleryWorker

# Перезапустить
nssm restart CeleryWorker

# Удалить службу
nssm remove CeleryWorker
```

### Преимущества:
- ✅ Запускается автоматически при старте Windows
- ✅ Перезапуск при сбоях
- ✅ Управление через Services.msc
- ✅ Логи в файл

### Недостатки:
- ⚠️ Сложнее настроить
- ⚠️ Redis/Django отдельно

---

## ✅ ВАРИАНТ 4: Supervisor (только Linux)

Если разворачиваете на Linux сервере:

```bash
# /etc/supervisor/conf.d/celery.conf
[program:celery]
command=/path/to/venv/bin/celery -A app worker --loglevel=info
directory=/path/to/project/Folder
user=www-data
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/celery.log
```

```bash
# Перезапустить
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start celery
```

---

## ✅ ВАРИАНТ 5: Systemd (только Linux)

```bash
# /etc/systemd/system/celery.service
[Unit]
Description=Celery Worker
After=network.target

[Service]
Type=forking
User=www-data
Group=www-data
WorkingDirectory=/path/to/project/Folder
ExecStart=/path/to/venv/bin/celery -A app worker --loglevel=info --detach
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable celery
sudo systemctl start celery
```

---

## 🎯 КАКОЙ ВЫБРАТЬ?

### Для разработки (локально):
```
1. НЕ запускать Celery вообще (работает через fallback)
2. Запускать вручную когда нужно (для тестирования скорости)
3. BAT-файл (двойной клик)
```

### Для production (сервер):
```
1. Docker Compose ✅ (самый простой и надёжный)
2. Systemd (если Linux без Docker)
3. Supervisor (если нужны сложные сценарии)
```

### Для Windows (локальный сервер):
```
1. Docker Compose ✅
2. NSSM (Windows Service)
3. Task Scheduler
```

---

## 📊 СРАВНЕНИЕ

| Метод | Автозапуск | Перезапуск | Сложность | Рекомендация |
|-------|------------|------------|-----------|--------------|
| **Docker Compose** | ✅ | ✅ | Низкая | ⭐⭐⭐⭐⭐ Production |
| **Task Scheduler** | ✅ | ❌ | Средняя | ⭐⭐⭐ Development |
| **NSSM** | ✅ | ✅ | Средняя | ⭐⭐⭐⭐ Windows Server |
| **Вручную** | ❌ | ❌ | Низкая | ⭐⭐ Testing |
| **Systemd** | ✅ | ✅ | Средняя | ⭐⭐⭐⭐ Linux Server |

---

## 🎬 ЧТО ДЕЛАТЬ СЕЙЧАС?

### Для локальной разработки:
```powershell
# Вариант А: Вообще не запускать (работает медленно, но работает)
python manage.py runserver

# Вариант Б: Двойной клик на start_celery.bat (когда нужна скорость)
# Или просто команда в терминале:
cd C:\allcodes\ad\Folder
celery -A app worker --loglevel=info --pool=solo
```

### Для продакшена:
```powershell
# Используйте Docker Compose ✅
docker compose up -d

# Всё запустится автоматически!
```

---

## ❓ ЧАСТО ЗАДАВАЕМЫЕ ВОПРОСЫ

### Q: Что если забуду запустить Celery?
**A:** Ничего страшного! Система работает в fallback режиме (синхронно, медленно).

### Q: Celery упал, что делать?
**A:** 
- **Docker:** `docker compose restart celery`
- **Вручную:** Перезапустить командой
- **NSSM:** `nssm restart CeleryWorker`

### Q: Как узнать что Celery работает?
**A:**
```powershell
# Проверить процессы
celery -A app inspect active

# Или через Flower
http://localhost:5555
```

### Q: Нужен ли Celery для маленького проекта?
**A:** Нет! Для <10 документов/день можно работать без Celery (синхронно).

### Q: Celery съедает много памяти?
**A:** 
- Один worker: ~150-300 MB RAM
- Docker контейнер: ~200-400 MB RAM
- Для 4-8 воркеров: ~500 MB - 1 GB RAM

---

## 🔧 МОНИТОРИНГ

### Проверить что Celery работает:

```powershell
# Активные задачи
celery -A app inspect active

# Статистика
celery -A app inspect stats

# Зарегистрированные задачи
celery -A app inspect registered
```

### Flower (веб-интерфейс):
```powershell
celery -A app flower --port=5555
# Откройте: http://localhost:5555
```

---

## 🎉 РЕКОМЕНДАЦИЯ

### ДЛЯ ВАС (разработка):

**Сейчас:**
1. Оставьте Celery запущенным в терминале (уже работает ✅)
2. При перезагрузке системы используйте `start_celery.bat`

**В будущем (продакшен):**
```powershell
docker compose up -d
```

Готово! Больше вручную ничего не нужно! 🚀

---

## 📝 ИТОГ

Celery НЕ обязательно запускать каждый раз вручную:
- ✅ Docker Compose - автоматически
- ✅ Task Scheduler - при старте Windows
- ✅ NSSM - как служба Windows
- ✅ BAT-файл - двойной клик

**Выбирайте что удобнее!** 🎯
