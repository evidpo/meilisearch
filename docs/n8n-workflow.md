# n8n Workflow: Синхронизация Directual → Meilisearch

## 📋 Обзор

Workflow загружает курсы из Directual и синхронизирует с Meilisearch:
- Добавляет/обновляет активные курсы
- Удаляет курсы с direction_of_study = "УПРАЗДНЕНО"

---

## 🔧 Структура Workflow

```
┌─────────────────┐
│ Manual Trigger  │ (или Schedule Trigger)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  HTTP Request   │ → Directual API
│ (Get Courses)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Transform     │ → JavaScript
│ (Map Fields)    │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌───────┐ ┌───────┐
│ POST  │ │DELETE │
│Active │ │Deleted│
└───────┘ └───────┘
```

---

## 📦 Ноды

### 1. Trigger

**Варианты:**
- `Manual Trigger` — для ручного запуска
- `Schedule Trigger` — для автоматизации (настроить Cron)

```
Рекомендуемый Cron: 0 3 * * * (каждый день в 03:00)
```

### 2. HTTP Request — Directual API

```
Method: GET
URL: https://api.directual.com/good/api/v5/data/[APP_ID]/courses_for_meilisearch
Headers:
  - Authorization: Bearer [SESSION_ID] (если нужен)
Query Parameters:
  - pageSize: 500 (или настроить пагинацию)
```

**Response:**
```json
{
  "result": "ok",
  "payload": [
    {
      "id": "123",
      "code": "K-001",
      "course_name": "Охрана труда",
      "direction_of_study": "Охрана труда",
      "price": "5000",
      "hours": "72",
      "course_students_count": "1500",
      "pic1": "https://...",
      "url": "https://evidpo.ru/k/123"
    }
  ]
}
```

### 3. Transform for Meilisearch (Code Node)

```javascript
// Получаем данные из Directual
const items = $input.first().json.payload || [];

// Разделяем на активные и удалённые
const activeCourses = [];
const deletedIds = [];

items.forEach(c => {
  if (c.direction_of_study === 'УПРАЗДНЕНО') {
    deletedIds.push(c.id);
  } else {
    activeCourses.push({
      id: c.id,
      code: c.code || '',
      title: c.course_name || '',
      qualification: c.qualification || '',
      direction: c.direction_of_study || '',
      hours: parseInt(c.hours) || 0,
      days: parseInt(c.days) || 0,
      price: parseInt(c.price) || 0,
      students: parseInt(c.course_students_count) || 0,
      image: c.pic1 || '',
      url: c.url || ''
    });
  }
});

return {
  json: {
    activeCourses,
    deletedIds
  }
};
```

### 4. HTTP Request — POST Active Courses

```
Method: POST
URL: https://[MEILI_HOST]/indexes/courses/documents
Headers:
  - Authorization: Bearer [MASTER_KEY]
  - Content-Type: application/json
Body: {{ $json.activeCourses }}
```

### 5. HTTP Request — DELETE Obsolete Courses

```
Method: POST
URL: https://[MEILI_HOST]/indexes/courses/documents/delete-batch
Headers:
  - Authorization: Bearer [MASTER_KEY]
  - Content-Type: application/json
Body: {{ $json.deletedIds }}
```

---

## 🔑 Credentials

### Directual (если нужна авторизация)
```
Type: Header Auth
Name: Authorization
Value: Bearer [SESSION_ID]
```

### Meilisearch
```
Type: Header Auth
Name: Authorization
Value: Bearer [MASTER_KEY]
```

---

## 📊 Результаты последнего запуска

```
Task ID: 16
Активных курсов: 200
Удалённых: 1
Время выполнения: ~3 секунды
```

---

## ⚙️ Настройка автоматизации

### Добавить Schedule Trigger

1. Удалить Manual Trigger (или оставить для ручного запуска)
2. Добавить Schedule Trigger
3. Настроить:
   - Trigger Interval: Days
   - Days Between Triggers: 1
   - Trigger at Hour: 3
   - Trigger at Minute: 0

### Или использовать Cron Expression

```
Mode: Cron
Cron Expression: 0 3 * * *
```

---

## 🐛 Отладка

### Проверить что Directual отдаёт данные

```bash
curl -X GET 'https://api.directual.com/good/api/v5/data/[APP_ID]/courses_for_meilisearch?pageSize=10'
```

### Проверить что Meilisearch обновился

```bash
curl -X GET 'https://[MEILI_HOST]/indexes/courses/stats' \
  -H 'Authorization: Bearer [SEARCH_KEY]'
```

### Логи в n8n

```
1. Открыть workflow
2. Кликнуть на выполненную ноду
3. Посмотреть Input/Output данные
```

---

## 📝 TODO

- [ ] Настроить Schedule Trigger
- [ ] Добавить уведомления об ошибках (Telegram/Email)
- [ ] Инкрементальная синхронизация (только изменённые)
- [ ] Логирование в отдельную структуру

---

**Обновлено:** 2026-01-05
