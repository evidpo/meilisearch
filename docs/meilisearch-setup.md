# Настройка Meilisearch

> ✅ **Актуально (с 2026-06-16): Meilisearch self-hosted на VPS evidpo** через Docker + Traefik.
> Сервис `meilisearch` в `/root/docker-compose.yml` на сервере evidpo (31.128.43.138):
> образ `getmeili/meilisearch:v1.9`, том `${DATA_FOLDER}/meili_data`, внешний адрес
> `https://search.evidpo.ru` (TLS через Traefik, certresolver `mytlschallenge`),
> внутренний адрес для n8n — `http://meilisearch:7700`. `MEILI_MASTER_KEY` — в `/root/.env`.
> Раздел ниже про Railway оставлен как исторический (старый хостинг гасится после миграции).

## 🚀 Развёртывание на Railway (архив — больше не используется)

### 1. Создание сервиса

```
1. Railway → New Project → Deploy from Template
2. Выбрать "Meilisearch"
3. Дождаться деплоя (~2 минуты)
```

### 2. Настройка переменных

```
MEILI_MASTER_KEY = [сгенерировать случайный]
MEILI_ENV = production
```

### 3. Получение URL

```
После деплоя Railway выдаст URL вида:
https://[project-name]-production-[hash].up.railway.app
```

---

## 🔧 Настройка индекса

### Создание индекса

```bash
curl -X POST 'https://[MEILI_HOST]/indexes' \
  -H 'Authorization: Bearer [MASTER_KEY]' \
  -H 'Content-Type: application/json' \
  -d '{
    "uid": "courses",
    "primaryKey": "id"
  }'
```

### Настройка фильтруемых полей

```bash
curl -X PATCH 'https://[MEILI_HOST]/indexes/courses/settings' \
  -H 'Authorization: Bearer [MASTER_KEY]' \
  -H 'Content-Type: application/json' \
  -d '{
    "filterableAttributes": ["direction"]
  }'
```

### Настройка сортируемых полей

```bash
curl -X PATCH 'https://[MEILI_HOST]/indexes/courses/settings' \
  -H 'Authorization: Bearer [MASTER_KEY]' \
  -H 'Content-Type: application/json' \
  -d '{
    "sortableAttributes": ["price", "students"]
  }'
```

### Показать все поля в ответе

```bash
curl -X PATCH 'https://[MEILI_HOST]/indexes/courses/settings' \
  -H 'Authorization: Bearer [MASTER_KEY]' \
  -H 'Content-Type: application/json' \
  -d '{
    "displayedAttributes": ["*"]
  }'
```

---

## 🔑 Генерация API ключей

### Получить Search-only ключ

```bash
curl -X GET 'https://[MEILI_HOST]/keys' \
  -H 'Authorization: Bearer [MASTER_KEY]'
```

Ответ содержит `Default Search API Key` — использовать в браузере.

### Создать кастомный ключ (опционально)

```bash
curl -X POST 'https://[MEILI_HOST]/keys' \
  -H 'Authorization: Bearer [MASTER_KEY]' \
  -H 'Content-Type: application/json' \
  -d '{
    "description": "evidpo.ru search key",
    "actions": ["search"],
    "indexes": ["courses"],
    "expiresAt": null
  }'
```

---

## 📊 Полезные команды

### Статистика индекса

```bash
curl -X GET 'https://[MEILI_HOST]/indexes/courses/stats' \
  -H 'Authorization: Bearer [SEARCH_KEY]'
```

### Посмотреть документы

```bash
curl -X GET 'https://[MEILI_HOST]/indexes/courses/documents?limit=10' \
  -H 'Authorization: Bearer [SEARCH_KEY]'
```

### Тестовый поиск

```bash
curl -X POST 'https://[MEILI_HOST]/indexes/courses/search' \
  -H 'Authorization: Bearer [SEARCH_KEY]' \
  -H 'Content-Type: application/json' \
  -d '{"q": "охрана труда", "limit": 5}'
```

### Удалить все документы

```bash
curl -X DELETE 'https://[MEILI_HOST]/indexes/courses/documents' \
  -H 'Authorization: Bearer [MASTER_KEY]'
```

### Удалить конкретные документы

```bash
curl -X POST 'https://[MEILI_HOST]/indexes/courses/documents/delete-batch' \
  -H 'Authorization: Bearer [MASTER_KEY]' \
  -H 'Content-Type: application/json' \
  -d '["id1", "id2", "id3"]'
```

---

## 📝 Текущие настройки проекта

```
Host: https://search.evidpo.ru
Index: courses
Documents: 8060

filterableAttributes: ["direction"]
sortableAttributes: ["price", "students"]
displayedAttributes: ["*"]
```

---

## ⚠️ Важно

1. **Master Key** — только для администрирования, НЕ использовать в браузере
2. **Search Key** — безопасно использовать в браузере
3. После публикации репозитория — сменить все ключи!

---

**Обновлено:** 2026-01-05
