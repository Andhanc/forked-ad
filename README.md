# 🎓 Система антиплагиата БГУИР

Веб-приложение для проверки документов на плагиат с использованием продвинутых алгоритмов машинного обучения.

---

## ✨ Возможности

- 📄 **Загрузка PDF и DOCX документов** с автоматическим извлечением текста
- 📊 **Извлечение таблиц** из документов с сохранением структуры
- 🤖 **Продвинутый анализ плагиата** с множественными метриками (5+ методов)
- 🚀 **Асинхронная обработка** через Celery для мгновенного отклика
- 🧠 **Векторизация текстов** с использованием sentence-transformers
- 🔍 **Детальный анализ схожести**: shingles, word, char, sentence similarity
- 🎚️ **Уровни риска**: very_low → very_high
- 📈 **Мониторинг** очередей через Flower
- 💾 **Redis кэширование** для ускорения повторных вычислений
- 🔐 **LDAP аутентификация** для интеграции с корпоративной системой
- 🎨 **Современный UI** с live-обновлениями и прогресс-барами

---

## 🏗️ Технологический стек

### Backend:
- **Django 4.2.9** - веб-фреймворк
- **PostgreSQL + pgvector** - база данных с векторным расширением
- **Celery** - асинхронная обработка задач
- **Redis** - брокер сообщений и кэш
- **Gunicorn** - WSGI сервер

### ML/NLP:
- **sentence-transformers 2.2.2** - векторизация текстов
- **torch 2.5.1** - backend для ML моделей
- **pdfminer.six** - извлечение текста из PDF
- **python-docx 1.1.0** - извлечение текста из DOCX
- **numpy 2.0.2** - математические вычисления

### Мониторинг:
- **Flower 2.0.1** - веб-интерфейс для Celery
- **Django Debug Toolbar** - отладка (только dev)

---

## 🚀 Быстрый старт (Docker)

### 1. Клонируйте репозиторий:
```bash
git clone https://github.com/andrei-shemerei/Plagiarism_detection_bsuir
cd Plagiarism_detection_bsuir
```

### 2. Создайте .env файл:
```bash
cp env.template .env
```

Отредактируйте `.env` (минимум):
```env
POSTGRES_DB=plagiarism_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your-password
SECRET_KEY=generate-new-secret-key
DEBUG=False
```

### 3. Запустите:
```bash
docker compose up -d --build
```

### 4. Создайте суперпользователя:
```bash
docker compose exec web python manage.py createsuperuser
```

### 5. Откройте в браузере:
- **Приложение:** http://localhost:8080
- **Flower (мониторинг):** http://localhost:5555

---

## 🖥️ Локальная разработка

### Требования:
- Python 3.11.5
- PostgreSQL 16.4
- Redis (опционально)

### Установка:
```bash
# 1. Создать виртуальное окружение
python -m venv venv
venv\Scripts\activate  # Windows
source venv/bin/activate  # Linux/Mac

# 2. Установить зависимости
pip install -r requirements.txt

# 3. Настроить переменные окружения
# Создайте .env или установите через систему

# 4. Применить миграции
cd Folder
python manage.py migrate

# 5. Создать суперпользователя
python manage.py createsuperuser

# 6. Запустить сервер
python manage.py runserver
```

Без Redis система работает в синхронном режиме (медленнее, но функционально).

---

## 📋 Структура проекта

```
Plagiarism_detection_bsuir/
├── Folder/                         # Django проект
│   ├── app/                        # Настройки приложения
│   │   ├── settings.py             # Конфигурация (env-based)
│   │   ├── celery.py               # Настройки Celery
│   │   └── urls.py                 # Главные маршруты
│   ├── documents/                  # Основное приложение
│   │   ├── detectors/              # Детекторы плагиата
│   │   │   ├── base_detector.py
│   │   │   └── advanced_detector.py
│   │   ├── management/commands/    # CLI команды
│   │   ├── migrations/             # Миграции БД
│   │   ├── templates/              # HTML шаблоны
│   │   ├── models.py               # Модели данных
│   │   ├── views.py                # Контроллеры
│   │   ├── tasks.py                # Celery задачи
│   │   ├── processing.py           # Синхронная обработка
│   │   ├── forms.py                # Формы
│   │   ├── docx_extractor.py       # Извлечение текста из DOCX
│   │   └── utils_cache.py          # Redis кэширование
│   ├── users/                      # Управление пользователями
│   ├── static/                     # Статические файлы
│   └── media/                      # Загруженные документы
├── Dockerfile                      # Docker образ
├── docker-compose.yml              # Оркестрация сервисов
├── entrypoint.sh                   # Скрипт инициализации
├── requirements.txt                # Python зависимости
├── env.template                    # Шаблон переменных окружения
├── DEPLOYMENT.md                   # Полная инструкция по деплою
├── DOCKER_READY.md                 # Чек-лист готовности
├── QUICK_START.md                  # Быстрый старт
├── DOCX_SUPPORT.md                 # Поддержка DOCX ✅
└── IMAGE_PLAGIARISM_GUIDE.md       # План проверки изображений 📋

Документация:
├── ASYNC_SETUP.md                  # Настройка Celery
└── LOCAL_SETUP.md                  # Локальная разработка
```

---

## 🔧 Полезные команды

### Docker:
```bash
# Запуск
docker compose up -d

# Остановка
docker compose down

# Логи
docker compose logs -f celery

# Миграции
docker compose exec web python manage.py migrate

# Пересчёт документов
docker compose exec web python manage.py reprocess_documents --all
```

### Локально:
```bash
# Сборка статики
python Folder/manage.py collectstatic

# Celery worker (нужен Redis)
celery -A Folder.app worker --loglevel=info --pool=solo

# Flower мониторинг
celery -A Folder.app flower
```

---

## 📊 Алгоритм проверки

### 1. Извлечение текста
- **PDF** → текст через pdfminer.six
- **DOCX** → текст + таблицы через python-docx
- **Таблицы** → сохраняются со структурой

### 2. Векторизация
Текст → эмбеддинг (384-мерный вектор) через sentence-transformers

### 3. Поиск похожих
Косинусное сходство векторов → топ-N кандидатов

### 4. Детальный анализ
- Shingles (1/3/5-граммы)
- Word similarity (уникальные слова)
- Character similarity (посимвольно)
- Sentence similarity (по предложениям)

### 5. Расчёт оригинальности
Минимальная оригинальность среди всех похожих документов (0-100%)

---

## 🎯 Workflow пользователя

1. **Загрузка** → Документ сохраняется с status="Проверен"
2. **Обработка** → Celery извлекает текст, векторизует, анализирует (асинхронно)
3. **Результат** → Оригинальность отображается в кабинете
4. **Отправка на защиту** → Кнопка меняет status="На защите"
5. **Оценка** → Преподаватель ставит "Зачтен"/"Не зачтен"

---

## 🔒 Безопасность

- Все секреты в переменных окружения
- DEBUG=False в продакшене
- ALLOWED_HOSTS фильтрация
- Session/CSRF cookies secure (с HTTPS)
- Персистентные volumes для данных

---

## 📈 Производительность

- **Время отклика:** <1 сек (вместо 30-60 сек)
- **Параллелизм:** 4-8 документов одновременно
- **Кэширование:** векторы (1 час), результаты (2 часа)
- **Retry:** автоматически 3 попытки при сбоях

---

## 🐛 Решение проблем

См. **DEPLOYMENT.md** раздел "Решение проблем"

---

## 📚 Документация

### Деплой:
- **DEPLOYMENT.md** - полная инструкция по деплою
- **QUICK_START.md** - быстрый старт
- **DOCKER_READY.md** - чек-лист готовности к продакшену
- **ASYNC_SETUP.md** - настройка асинхронной обработки
- **LOCAL_SETUP.md** - локальная разработка без Docker

### Функциональность:
- **DOCX_SUPPORT.md** - поддержка DOCX файлов (реализовано ✅)
- **IMAGE_PLAGIARISM_GUIDE.md** - детальный план проверки изображений (план 📋)

---

## 👥 Авторы

Проект разработан в БГУИР для автоматизации проверки студенческих работ на плагиат.

---

## 📄 Лицензия

Все права защищены © БГУИР 2025