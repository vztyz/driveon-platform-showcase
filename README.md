

# 🚗 DRIVE ON — B2B/B2C-платформа цифровых автопрошивок и калибровок ЭБУ

> **Showcase-репозиторий (Commercial / Closed Source).**  
> Исходный код коммерческого проекта не публикуется. Здесь представлены архитектура, технологический стек, реализованные инженерные решения, скриншоты и видеодемонстрация работы системы.

![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-5.0-092E20?logo=django&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1?logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white)
![Celery](https://img.shields.io/badge/Celery-5.3-37814A?logo=celery&logoColor=white)
![S3](https://img.shields.io/badge/Object_Storage-S3_%2F_Boto3-FF9900?logo=amazons3&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/TailwindCSS-Dark_UI-06B6D4?logo=tailwindcss&logoColor=white)

---

## 🎬 Видеодемонстрация (27 сек)

> Демонстрация пользовательского пути (тёмная тема, cookie-баннер по 152-ФЗ, каталог) и кастомной панели администратора (аналитика, нодовый редактор, проводник БД, распознаватель ЭБУ, Django Jazzmin).

📹Смотреть демо-видео 

https://github.com/user-attachments/assets/0b6733e2-6fe7-4b84-bf68-4377771187c5

---

## 📸 Интерфейсы (Dark Mode)

### Витрина и каталог
![Витрина магазина](assets/01_storefront.png)

### Кастомный дашборд администратора
![Дашборд аналитики](assets/02_dashboard.png)

### Интерактивный нодовый редактор каталога
![Нодовый редактор](assets/03_node_editor.png)

### Проводник базы данных и массовое управление прошивками
![Проводник БД](assets/04_db_explorer.png)

### Сервис распознавания блоков управления (ЭБУ Bosch)
![Распознаватель ЭБУ](assets/05_ecu_recognizer.png)

---

## 🏗 Архитектура системы

```mermaid
flowchart LR
    U[Пользователь / Покупатель] --> C[Caddy Reverse Proxy / HTTPS]
    C --> D[Django 5 + Gunicorn]
    D --> PG[(PostgreSQL 15)]
    D --> R[(Redis: сессии, кэш, брокер)]
    D --> S3[(S3 Object Storage\nSigned URLs)]
    D --> CW[Celery Workers]
    CW --> TG[Telegram Bot API\nуведомления и заказы]
    CW --> PAY[Платёжные шлюзы\nYooKassa / Tochka / CryptoPay]
    D --> ECU[Модуль распознавания ЭБУ\nлокальная БД + парсеры]
```

---

## ⚙️ Технологический стек и ключевые модули

### Backend & Хранение данных
- **Django 5.0 + Gunicorn** — монолитное веб-приложение с SSR, модулями `shop` (магазин, заказы, DRM, калибровки) и `bosch_ecu` (распознавание блоков управления).
- **PostgreSQL 15** — основная БД в продакшене (`dj-database-url`), локальная `SQLite` для изолированной разработки.
- **Redis + Celery** — асинхронная очередь задач (доставка прошивок, агрегированные дайджесты продаж, уведомления), хранение сессий (`cached_db`) и FSM-состояний Telegram-бота.
- **S3 (`django-storages` + `boto3`)** — хранение тяжёлых бинарных файлов прошивок в приватном бакете и выдача временных подписанных ссылок (`AWS_QUERYSTRING_AUTH = True`, TTL 3600 с).

### Frontend & UX
- **TailwindCSS + Alpine.js** — адаптивный интерфейс с переключением светлой/тёмной темы (`dos_theme`), корзиной в `localStorage` и интерактивным каталогом.
- **Кастомный нодовый редактор** — визуальный граф категорий и прошивок на SVG/HTML-холсте (масштабирование, панорамирование, редактирование связей).
- **Проводник БД и ZIP-импорт** — пакетная загрузка, перенос и редактирование позиций без входа в стандартную админку.

### Безопасность и соответствие 152-ФЗ
- **2FA / TOTP (`django-otp`) + кастомный `AdminGateMiddleware`** — дополнительный шлюз перед доступом к админ-панелям.
- **Защита от брутфорса (`django-axes`)** — блокировка после серии неудачных попыток входа.
- **Content Security Policy (`CSPMiddleware`)** — генерация `nonce` на каждый запрос и строгие директивы для скриптов/медиа.
- **Куки и оферта (152-ФЗ)** — баннер согласия на аналитику и `ConsentLog` с фиксацией версии оферты, IP и User-Agent при регистрации.
- **Изоляция контейнеров** — запуск в Docker под непривилегированным пользователем (`appuser`).

### Интеграции
- **Платёжные шлюзы:** YooKassa, Точка Банк, CryptoPay (webhook-обработчики с верификацией).
- **Telegram-бот (`pyTelegramBotAPI`):** мгновенные уведомления администраторов о заказах и заявках на индивидуальную калибровку, ежедневные/недельные дайджесты.
- **Распознаватель ЭБУ Bosch:** поиск по номеру блока с агрегацией данных из локальной базы и внешних источников.
