# 🎯 Правила работы с проектом evidpo-search

## 📋 Источники истины

1. **`.trae/rules/memory-bank/`** — техническая документация:
   - `brief.md` — суть проекта (2-3 предложения)
   - `stack.md` — технологии, URLs, ключи, архитектура
   - `constraints.md` — ограничения и особенности
   - `features.md` — функции с статусами реализации

2. **`README.md`** — описание проекта, быстрый старт
3. **`TASKS.md`** — задачи по приоритету
4. **`src/`** — исходный код виджета

---

## 🚀 Процесс работы

### ✅ Начало задачи

```
1. Прочитай brief.md (суть проекта)
2. Прочитай constraints.md (ограничения)
3. Если работа с:
   - Meilisearch/n8n → читай stack.md
   - Виджетом → открой src/evidpo-search-overlay.html
4. Проверь TASKS.md — есть ли похожая задача
5. Предложи план (3-5 шагов)
6. Получи подтверждение → начинай
```

### 🔨 Выполнение

```
1. Код виджета — только Vanilla JS (без npm)
2. Стили — inline в <style> блоке
3. Используй IIFE для изоляции
4. Комментируй сложную логику
5. Тестируй на всех брейкпоинтах
```

### ✅ Завершение задачи

```
1. Покажи что изменилось
2. Обнови TASKS.md (статус → ✅)
3. Обнови features.md если нужно
4. Создай changelog (см. ниже)
5. Предложи следующие шаги
```

---

## 📝 Changelogs

### После КАЖДОЙ задачи создавай changelog:

```
changelogs/changelog_YYYY-MM-DD_HH-MM.md
```

### Шаблон changelog:

```markdown
# Changelog YYYY-MM-DD_HH-MM

## [Краткое название изменения]

### 🎯 Что сделано

[Описание в 2-3 предложениях]

---

### 🔄 Изменения

**[Категория 1]:**
- Изменение 1
- Изменение 2

**[Категория 2]:**
- Изменение 1

---

### 📁 Затронутые файлы

- `path/to/file1.html` — что изменилось
- `path/to/file2.md` — что изменилось

---

### 📝 Примечания

[Дополнительные заметки, если нужны]

---

**Автор:** [имя]  
**Дата:** YYYY-MM-DD
```

### Когда создавать changelog:

✅ Новая функция в виджете
✅ Изменение настроек Meilisearch  
✅ Изменение n8n workflow
✅ Исправление бага
✅ Изменение архитектуры

❌ Мелкие правки документации
❌ Обновление комментариев

---

## 📁 Структура проекта

```
evidpo-search/
├── .trae/rules/
│   ├── project_rules.md          # Это — правила работы
│   └── memory-bank/
│       ├── brief.md              # Суть проекта
│       ├── stack.md              # Технологии, URLs, ключи
│       ├── constraints.md        # Ограничения
│       └── features.md           # Функции с статусами
├── src/
│   └── evidpo-search-overlay.html  # Виджет поиска
├── docs/
│   ├── meilisearch-setup.md      # Настройка Meilisearch
│   └── n8n-workflow.md           # Описание workflow
├── changelogs/                    # История изменений
│   └── changelog_YYYY-MM-DD_HH-MM.md
├── README.md                      # Описание проекта
├── TASKS.md                       # Задачи
└── .gitignore
```

---

## 🔧 Ключевые команды

### Meilisearch API

```bash
# Проверить статус индекса
curl -X GET 'https://search.evidpo.ru/indexes/courses/stats' \
  -H 'Authorization: Bearer SEARCH_KEY'

# Поиск
curl -X POST 'https://search.evidpo.ru/indexes/courses/search' \
  -H 'Authorization: Bearer SEARCH_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"q": "охрана труда", "limit": 10}'

# Обновить настройки (только с Master Key!)
curl -X PATCH '.../indexes/courses/settings' \
  -H 'Authorization: Bearer MASTER_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"filterableAttributes": ["direction"]}'
```

### Тестирование виджета

```bash
# Открыть локально
open src/evidpo-search-overlay.html

# Или через Live Server в VS Code
```

---

## ❌ Что НЕ делать

```
❌ Использовать Master Key в браузере
❌ Добавлять npm зависимости в виджет
❌ Менять структуру индекса без обновления n8n workflow
❌ Удалять курсы напрямую в Meilisearch (только через n8n)
❌ Забывать про мобильную адаптивность
```

## ✅ Что МОЖНО делать

```
✅ Изменять стили виджета
✅ Добавлять новые функции в JS
✅ Обновлять маппинг directionToQuery
✅ Тестировать API через curl
✅ Запускать n8n workflow вручную
```

---

## 💡 Частые задачи

### "Добавить новое направление в маппинг"

```javascript
// В src/evidpo-search-overlay.html найти:
const directionToQuery = {
  // Добавить новую строку:
  'Новое направление': 'поисковый запрос',
};
```

### "Изменить количество карточек"

```javascript
// Найти константу:
const ITEMS_PER_PAGE = 6;  // Изменить на нужное
```

### "Добавить новое поле в карточку"

```javascript
// 1. Убедиться что поле есть в Meilisearch
// 2. Добавить в n8n Transform если нужно
// 3. Обновить renderResults() в виджете
```

### "Перезапустить синхронизацию"

```
1. Открыть n8n
2. Найти workflow "Sync Directual → Meilisearch"
3. Нажать "Execute Workflow"
4. Проверить результат в Meilisearch
```

---

## 🎯 Проверочный список

### Перед изменением виджета
```
□ Прочитал constraints.md?
□ Код работает без npm?
□ Протестировал на мобильном (< 1024px)?
□ Не использую Master Key?
```

### После изменения
```
□ Обновил TASKS.md?
□ Обновил features.md (если новая функция)?
□ Создал changelog_YYYY-MM-DD_HH-MM.md?
□ Виджет работает на всех брейкпоинтах?
□ Сделал коммит на русском языке?
```

### Git коммиты
```
✅ Всегда коммитить на русском языке
✅ Коммитить сразу после завершения задачи
✅ Формат: "docs/feat/fix(scope): описание"
```

---

## 🔐 Безопасность

### ⚠️ КРИТИЧНО после публикации репозитория:

```
1. Ротировать MEILI_MASTER_KEY в /root/.env на сервере evidpo
2. Сгенерировать новый Search Key
3. Обновить ключ в виджете
4. Обновить ключ в n8n workflow
```

---

**Версия:** 1.0  
**Дата:** 2026-01-05
