# 🔍 Поисковый виджет evidpo.ru

Полноэкранный оверлей и виджет рекоммендаций поиска курсов для сайта evidpo.ru на Creatium.

**Поисковый движок:** Meilisearch (self-hosted, search.evidpo.ru)  
**Синхронизация:** n8n + Directual  

---

## ✨ Возможности

- 🔎 Мгновенный поиск с подсветкой совпадений
- 📂 Динамические категории (топ-10 по популярности)
- 🔥 "Часто ищут" — топ-4 направления
- 📱 Адаптивный дизайн (6 брейкпоинтов)
- ♾️ Пагинация "Показать ещё"
- 🖼️ Карточки курсов с изображениями

---

## 🔧 Технологии

| Компонент | Технология |
|-----------|------------|
| Поиск | Meilisearch v1.9 |
| Хостинг поиска | Self-hosted (Docker на VPS evidpo, `search.evidpo.ru`) |
| Синхронизация | n8n |
| Источник данных | Directual |
| Сайт | Creatium |
| Виджет | Vanilla HTML/CSS/JS |

---

## 📊 API

### Поиск курсов

```bash
curl -X POST 'https://search.evidpo.ru/indexes/courses/search' \
  -H 'Authorization: Bearer 481488201223fa89d66079b70b6e6d1b5a428fe3d7a9d13f29ce508d72edf25a' \
  -H 'Content-Type: application/json' \
  -d '{
    "q": "охрана труда",
    "limit": 10,
    "attributesToHighlight": ["title"]
  }'
```

### Популярные курсы

```bash
curl -X POST '.../indexes/courses/search' \
  -H 'Authorization: Bearer SEARCH_KEY' \
  -d '{
    "q": "",
    "limit": 6,
    "sort": ["students:desc"]
  }'
```

---

## 🔄 Синхронизация данных

Данные синхронизируются из Directual через n8n workflow:

```
Directual API → n8n Transform → Meilisearch
```

**Запуск вручную:**
1. Открыть n8n
2. Найти workflow "Sync Directual → Meilisearch"
3. Execute Workflow

**Автоматически:** Настроен Cron триггер (TODO)

---

## 🔐 Безопасность

⚠️ **Правила:**

1. `MEILI_MASTER_KEY` хранится только в `/root/.env` на сервере evidpo — никогда в репо/браузере
2. В виджетах (`src/`) используется только read-only **Search Key**
3. При ротации ключей: новый master в `/root/.env` + `docker compose up -d meilisearch`, новый search-key → виджеты + n8n
4. Search Key в `src/*.html` безопасен для публикации (только чтение индекса `courses`)

---

## 📝 Документация

- [Настройка Meilisearch](docs/meilisearch-setup.md)
- [n8n Workflow](docs/n8n-workflow.md)
- [Ограничения](/.trae/rules/memory-bank/constraints.md)
- [Функции](/.trae/rules/memory-bank/features.md)

---

## 🤝 Разработка

Перед работой прочитай:
1. `.trae/rules/project_rules.md` — процесс работы
2. `.trae/rules/memory-bank/constraints.md` — ограничения
3. `TASKS.md` — текущие задачи

---

# 🔍 evidpo-search

Поисковый виджет для evidpo.ru — мгновенный поиск по курсам.

## Как это работает

```
┌──────────────────────────────────────────────────────────────────┐
│                         СИНХРОНИЗАЦИЯ                            │
│                                                                  │
│   Directual ──[unsynced=true]──► n8n ──► Meilisearch             │
│      │                            │           │                  │
│      │   При изменении курса      │           │                  │
│      │   ставится unsynced=true   │           │                  │
│      │                            │           │                  │
│      ◄────[unsynced=false]────────┘           │                  │
│         После синхронизации                   │                  │
│         флаг сбрасывается                     │                  │
└──────────────────────────────────────────────────────────────────┘
                                                │
                                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                          ПОИСК                                   │
│                                                                  │
│   Пользователь ──► Виджет ──► Meilisearch (~5ms) ──► Результаты  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

## Ключевые файлы

| Файл | Что делает |
|------|------------|
| `src/evidpo-search-overlay.html` | Виджет поиска (вставлять в body Creatium) |
| `src/evidpo-recommendations.html` | Блок "Похожие курсы" |

## Логика синхронизации (n8n)

**Триггер:** каждые 5 минут (Schedule Trigger)

**Шаги:**
1. Запрос курсов с `unsynced=true` из Directual
2. Если `direction_of_study = "УПРАЗДНЕНО"` → удалить из Meilisearch
3. Остальные → добавить/обновить в Meilisearch
4. Сбросить `unsynced=false` в Directual

**Результат:** синхронизируются только изменённые курсы, не все 200+.

## Быстрый старт

1. **Виджет поиска** → скопировать `src/evidpo-search-overlay.html` в Creatium (Свой код → body)
2. **Кнопка поиска** → добавить в нужное место: `<button data-evidpo-search>Поиск</button>`
3. **n8n workflow** → импортировать JSON, настроить credentials

## MCP (Model Context Protocol)

Workflow доступен как MCP-инструмент для Claude и других AI.

**Server URL:**
```
https://n8n.evidpoai.ru/mcp-server/http
```

**Подключение (Claude Desktop / Cursor / etc):**
```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": [
        "-y",
        "supergateway",
        "--streamableHttp",
        "https://n8n.evidpoai.ru/mcp-server/http",
        "--header",
        "authorization:Bearer <YOUR_ACCESS_TOKEN>"
      ]
    }
  }
}
```

---

## API ключи

```
Meilisearch Host: https://search.evidpo.ru (см. .trae/rules/memory-bank/stack.md)
Search Key: безопасен для браузера (только чтение)
Master Key: только в n8n! (запись/удаление)
```

---

Подробная документация: `.trae/rules/memory-bank/`
