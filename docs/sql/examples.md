# Примеры запросов

В этом разделе — практические примеры запросов к таблице `tickets` (заявки в службу поддержки).

## Структура таблицы `tickets`

| Столбец | Тип | Описание |
| :--- | :--- | :--- |
| `ticket_id` | integer | Уникальный номер заявки |
| `created_at` | timestamp | Дата и время создания |
| `closed_at` | timestamp | Дата и время закрытия |
| `status` | text | Статус (`new`, `in_progress`, `resolved`, `closed`) |
| `priority` | text | Приоритет (`low`, `medium`, `high`) |
| `client_id` | integer | ID клиента |
| `response_time_minutes` | integer | Время до первого ответа (в минутах) |
| `resolution_time_minutes` | integer | Время до закрытия (в минутах) |

## Пример 1. Сколько заявок в каждом статусе?

```sql
SELECT 
    status,
    COUNT(ticket_id) AS quantity_tickets
FROM tickets
GROUP BY status
ORDER BY status
```

**Что делает:** считает количество заявок для каждого статуса.

**Зачем нужно:** понять распределение нагрузки и увидеть, сколько заявок «зависло» в каждом статусе.

## Пример 2. Среднее время решения по приоритетам

```sql
SELECT 
    priority,
    ROUND(AVG(resolution_time_minutes - response_time_minutes), 2) AS avg_resolution_time
FROM tickets
GROUP BY priority
ORDER BY avg_resolution_time DESC
```

**Что делает:** считает среднее время решения заявки для каждого приоритета.

**Зачем нужно:** понять, какие приоритеты обрабатываются дольше всего.

## Пример 3. Сколько заявок закрывалось каждый день за последнюю неделю?

```sql
SELECT 
    COUNT(ticket_id) AS quantity_tickets,
    DATE_TRUNC('day', closed_at) AS closed_date
FROM tickets
WHERE closed_at >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY DATE_TRUNC('day', closed_at)
ORDER BY closed_date
```

**Что делает:** показывает ежедневную динамику закрытия заявок за последние 7 дней.

**Зачем нужно:** увидеть, в какие дни было больше всего закрытий, и оценить эффективность команды.

## Пример 4. Топ-5 клиентов по количеству заявок

```sql
SELECT 
    client_id,
    COUNT(ticket_id) AS total_tickets
FROM tickets
GROUP BY client_id
ORDER BY total_tickets DESC
LIMIT 5
```

**Что делает:** находит 5 клиентов, которые создали больше всего заявок.

**Зачем нужно:** понять, кто из клиентов требует больше всего внимания.

## Пример 5. Заявки с высоким приоритетом, которые ещё не закрыты

```sql
SELECT 
    ticket_id,
    status,
    priority,
    created_at
FROM tickets
WHERE priority = 'high'
AND status != 'closed'
ORDER BY created_at ASC
```

**Что делает:** показывает все открытые заявки с высоким приоритетом, отсортированные по дате создания.

**Зачем нужно:** контролировать критичные заявки и не допускать их «зависания».

## Пример 6. Клиенты, у которых больше 10 заявок

```sql
SELECT 
    client_id,
    COUNT(ticket_id) AS total_tickets
FROM tickets
GROUP BY client_id
HAVING COUNT(ticket_id) > 10
ORDER BY total_tickets DESC
```

**Что делает:** находит клиентов с более чем 10 заявками.

**Зачем нужно:** выявить «активных» клиентов и понять, нужна ли им дополнительная поддержка.

## Что я вынесла

1. **SQL — это про логику, а не про зазубривание команд.** Важно понимать, что именно ты считаешь и зачем.
2. **Порядок блоков критичен.** `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `ORDER BY` → `LIMIT`.
3. **`WHERE` и `HAVING` — не одно и то же.** `WHERE` фильтрует строки, `HAVING` — группы.
4. **Даты — это отдельная тема.** `EXTRACT`, `DATE_TRUNC`, `CURRENT_DATE`, `INTERVAL` решают большинство задач.
5. **`ROUND` помогает** сделать результат читаемым.