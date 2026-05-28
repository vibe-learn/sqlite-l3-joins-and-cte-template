        # sqlite — JOIN-ы и CTE

        Homework-шаблон для урока **l3_joins_and_cte** (JOIN-ы и CTE) на платформе Vibe Learn.

        ## Что делать

        На Go с modernc.org/sqlite (без CGO) реализуй мини-аналитику с проверкой планов. Функции:
Setup(db) — создаёт orders/customers/regions и наполняет синтетикой; RevenueByRegion(db) —
выполняет трёхтабличный join + GROUP BY и возвращает []RegionRevenue; PlanHasScanJoin(db,
query) → bool — по EXPLAIN QUERY PLAN определяет, есть ли «опасный» SCAN внутренней таблицы
join (а не SEARCH); Descendants(db, rootID) — рекурсивным CTE возвращает всех потомков узла
в таблице-дереве. Тесты на :memory:-базе: проверяют корректность сумм по регионам; что без
индекса на orders(customer_id) PlanHasScanJoin=true, а после CREATE INDEX — false (внутренний
поиск стал SEARCH USING INDEX); и что рекурсивный CTE корректно обходит дерево.

## Контекст (из transfer-задачи урока)

Аналитическая фича на SQLite-базе (она же файл, который скачивают пользователи). Таблицы:
orders(id INTEGER PRIMARY KEY, customer_id INTEGER, total_cents INTEGER),
customers(id INTEGER PRIMARY KEY, region_id INTEGER, name TEXT),
regions(id INTEGER PRIMARY KEY, name TEXT). Объёмы: orders ~2 млн, customers ~200k, regions 50.
Отчёт «выручка по регионам» соединяет все три таблицы. Жалоба: на больших файлах отчёт
«висит» десятки секунд, хотя у пользователей с маленькими базами всё мгновенно.

Запрос:
```sql
SELECT r.name, sum(o.total_cents)
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN regions r ON c.region_id = r.id
GROUP BY r.name;
```

## Recap из урока

- **SQLite использует ТОЛЬКО nested loop** для join — нет hash join и merge join (в отличие от PostgreSQL). Цитата из доков: «SQLite implements joins as nested loops».
- Следствие: **индексы на join-колонках обязательны**. Без индекса соединение деградирует до **O(N×M)**; с индексом — O(N·log M). Спасательного hash join нет.
- Читай план join: `SEARCH` на внутренней таблице — хорошо; второй `SCAN` — тревога (O(N×M)). Иногда SQLite строит временный `AUTOMATIC INDEX` — лучше создать постоянный сам.
- JOIN-ы: INNER/LEFT/CROSS — давно; **RIGHT и FULL OUTER — только с 3.39.0 (2022)**. На старых сборках их может не быть.
- Продвинутый SQL есть: CTE (`WITH`), **рекурсивные CTE** для деревьев/графов, **оконные функции** (с 3.25.0). Subquery flattening вплавляет подзапросы/VIEW во внешний запрос ради индексов.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go`, прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - SQLite встроена — **никакого сервера и docker-compose**. БД это один файл (`DATABASE_PATH`) или `:memory:` в тестах. Драйвер `modernc.org/sqlite` — чистый Go, без CGO, так что CI собирается без компилятора C.

        ## Запуск

        ```bash
        # Прогнать тесты (бегут на :memory:-базе, ничего поднимать не нужно)
        go test ./...

        # Запустить main (создаёт схему, печатает marker; замени stub на реализацию)
        go run .

        # На файловой БД (нужно для WAL/concurrency-уроков):
        DATABASE_PATH=./app.db go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.
