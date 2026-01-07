# Технологический стек

## 🔧 Архитектура

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Directual     │────▶│      n8n        │────▶│   Meilisearch   │
│   (База курсов) │     │   (Синхронизация)│     │    (Поиск)      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                        │
                                                        ▼
                                                ┌─────────────────┐
                                                │    Creatium     │
                                                │  (HTML виджет)  │
                                                └─────────────────┘
```

---

## 🌐 Сервисы и URLs

### Meilisearch (Railway)
```
Host: https://getmeilimeilisearchv190-production-6123b.up.railway.app
Index: courses
Documents: 201

Master Key: 16mx5lg484k1bwyfamrxp6gql25stxu8
Search Key: b17c51453595ca37ff81b5c5b7ae8b7d0541d6b3e375fecd86831572dee58660
```

### n8n Workflow
```
Workflow: Sync Directual → Meilisearch
Trigger: Manual / Cron (настроить)
Task ID последнего запуска: 16
```

### Directual (источник данных)
```
API Endpoint: courses_for_meilisearch
Структура: courses2
Поля: id, code, course_name, qualification, direction_of_study, hours, days, price, course_students_count, pic1, url
```

### Creatium (сайт)
```
Production: https://evidpo.ru
Тестовая страница: https://evidpo.ru/search-test
Виджет: Встраивается через Custom HTML блок
```

---

## 📦 Индекс Meilisearch

### Поля документа
```javascript
{
  id: String,           // ID курса
  code: String,         // Код курса
  title: String,        // Название (course_name)
  qualification: String,// Квалификация
  direction: String,    // Направление (direction_of_study)
  hours: Number,        // Часы обучения
  days: Number,         // Дни обучения
  price: Number,        // Цена
  students: Number,     // Количество студентов (для сортировки)
  image: String,        // URL картинки (pic1)
  url: String           // URL страницы курса
}
```

### Настройки индекса
```bash
# Searchable attributes (по умолчанию все)
# Filterable attributes
filterableAttributes: ["direction", "code"]

# Sortable attributes
sortableAttributes: ["price", "students"]

# Displayed attributes
displayedAttributes: ["*"]
```

### Обновление filterableAttributes
```bash
curl -X PATCH 'https://getmeilimeilisearchv190-production-6123b.up.railway.app/indexes/courses/settings' \
  -H 'Authorization: Bearer MASTER_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"filterableAttributes": ["direction", "code"]}'
```

---

## 🛠 Технологии виджета

```
HTML5 + CSS3 + Vanilla JavaScript
- Без зависимостей
- IIFE для изоляции
- Адаптивный дизайн (6 брейкпоинтов)
- CSS переменные для кастомизации
```

### Брейкпоинты
```css
1400px+ — 4 колонки
1200px  — 3 колонки, узкий сайдбар
1024px  — мобильный layout (сайдбар горизонтально)
768px   — уменьшенные отступы
480px   — компактный вид
360px   — минимальный (карточки вертикально)
```

---

## 🔄 n8n Workflow

### Ноды
```
1. Manual Trigger / Schedule Trigger
2. HTTP Request → Directual API
3. Transform for Meilisearch (JS)
4. HTTP Request → Meilisearch POST /documents
5. [Опционально] Delete obsolete documents
```

### Transform логика
```javascript
// Разделение на активные и удалённые
const activeCourses = items.filter(c => c.direction_of_study !== 'УПРАЗДНЕНО');
const deletedIds = items.filter(c => c.direction_of_study === 'УПРАЗДНЕНО').map(c => c.id);

// Маппинг полей
activeCourses.map(c => ({
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
}));
```

---

**Последнее обновление:** 2026-01-05
