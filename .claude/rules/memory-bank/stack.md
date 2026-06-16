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

### Meilisearch (self-hosted на сервере evidpo, Docker + Traefik)
```
Host (внешний, для браузера):  https://search.evidpo.ru
Host (внутренний, для n8n):    http://meilisearch:7700   (docker-сеть root_default, без TLS)
Сервер:  evidpo (31.128.43.138), контейнер `meilisearch`, образ getmeili/meilisearch:v1.9
Том данных: ${DATA_FOLDER}/meili_data → /meili_data  (DATA_FOLDER=/root/n8n/)
TLS: Let's Encrypt через Traefik (certresolver mytlschallenge)
Index: courses
Documents: 8060

Master Key: <хранится в /root/.env на сервере evidpo (MEILI_MASTER_KEY), в репозиторий не коммитится>
Search Key (read-only, для виджетов): 481488201223fa89d66079b70b6e6d1b5a428fe3d7a9d13f29ce508d72edf25a
```

> ⚠️ Раньше Meilisearch жил на Railway. С 2026-06-16 перенесён на собственный VPS evidpo.
> Railway гасится после периода стабильной работы.

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

### Настройки индекса (актуально, зеркалировано с Railway 2026-06-16)
```bash
# Searchable attributes (порядок важен — влияет на ранжирование)
searchableAttributes: ["title", "full_title", "qualification", "direction", "topics", "code"]

# Filterable attributes
filterableAttributes: ["code", "direction", "id"]

# Sortable attributes
sortableAttributes: ["price", "students"]

# Displayed attributes
displayedAttributes: ["*"]

# Synonyms — 9 групп (настроены вручную, НЕ пересоздаются n8n!):
#   охрана труда → от, охраны труда, охране труда
#   педагог → учитель, преподаватель;  повышение квалификации → пк
#   пожарная безопасность → пб, птм;  электробезопасность → электрика
#   переподготовка, бухгалтер, воспитатель, модельер — см. /indexes/courses/settings
```

> ⚠️ При пересоздании индекса синонимы и searchable/filterable надо восстановить вручную
> (PATCH /indexes/courses/settings с Master Key) — n8n-синк льёт только документы.

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

**Последнее обновление:** 2026-06-16 (миграция Railway → self-hosted на evidpo)
