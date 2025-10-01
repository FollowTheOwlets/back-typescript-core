# SQL

<warning>
Все запросы вы можете протестировать запустив базу в докере. 
Прекрасно подойдет пример из конца топика про Docker. 

Перейдите на <a href="http://localhost:8082/login?next=/">http://localhost:8082</a>

Авторизуйтесь с помощью `admin@domain.com` и `adminpassword`

К базе в PGAdmin внутри докера можно будет подключиться по следующим параметрам
`host: postgres `
`port: 5432 `
`user: myuser `
`password: mypassword`
</warning>
<p>
    Язык
    <tooltip term="SQL">SQL</tooltip>
    декларативный: вы описываете
    <format style="bold">какие</format>
    данные нужны, а не
    <format style="bold">как</format>
    их получить. Базовый набор команд одинаков в большинстве СУБД.
</p>

## Основные команды SQL

<p>
    Команда
    <tooltip term="SELECT">SELECT</tooltip>
    выбирает столбцы из таблицы. Минимальная форма: указать столбцы и источник данных.
</p>

   ```sql
   SELECT column1, column2
   FROM tablename;
   ```

<p>
    <tooltip term="INSERT">INSERT</tooltip>
    добавляет новые записи. Порядок значений должен соответствовать перечисленным столбцам.
</p>
   ```sql
   INSERT INTO tablename (column1, column2) 
   VALUES ('value1', 'value2');
   ```

<p>
    <tooltip term="UPDATE">UPDATE</tooltip>
    изменяет существующие данные. Ограничивайте изменяемые строки условием
    <tooltip term="WHERE">WHERE</tooltip>
    .
</p>
   ```sql
   UPDATE tablename 
   SET column1 = 'new_value' 
   WHERE column2 = 'condition';
   ```

<p>
    <tooltip term="DELETE">DELETE</tooltip>
    удаляет строки, удовлетворяющие условию.
</p>
   ```sql
   DELETE FROM tablename 
   WHERE column1 = 'condition';
   ```

<p>
    <tooltip term="CREATE">CREATE</tooltip>
    создаёт объекты (таблицы, базы),
    <tooltip term="DROP">DROP</tooltip>
    — удаляет.
</p>
   ```sql
   CREATE TABLE tablename (
       column1 datatype,
       column2 datatype
   );
   ```

   ```sql
   DROP TABLE tablename;
   ```

<warning><p><code>DROP</code> необратим без резервной копии. Для безопасного удаления используйте <code>DROP
    TABLE IF EXISTS</code>.</p></warning>

<chapter title="Типы данных в PostgreSQL">
    <p>
        Корректный выбор типов повышает производительность и экономит место. В
        <tooltip term="PostgreSQL">PostgreSQL</tooltip>
        доступны:
    </p>
    <list>
        <li>
            <format style="bold">Числовые</format>: <code>smallint</code>, <code>integer</code>, <code>bigint</code>, <code>decimal</code>, <code>numeric</code>.
        </li>
        <li>
            <format style="bold">Строковые</format>: <code>varchar</code>, <code>text</code>, <code>char</code>.
        </li>
        <li>
            <format style="bold">Дата и время</format>: <code>date</code>, <code>time</code>, <code>timestamp</code>.
        </li>
        <li>
            <format style="bold">Логический</format>: <code>boolean</code>.
        </li>
        <li>
            <format style="bold">Структурированные</format>: <code>json</code>, <code>jsonb</code>, массивы <code>[]</code>.
        </li>
    </list>
    <tip>
        <p><code>jsonb</code> предпочтителен для индексации и поиска по ключам JSON.</p>
    </tip>
</chapter>

## Операции с данными

- **Фильтрация**: Используется оператор `WHERE`.
   ```sql
   SELECT * 
   FROM tablename 
   WHERE column1 = 'condition';
   ```

- **Сортировка**: Используется оператор `ORDER BY`.
   ```sql
   SELECT * 
   FROM tablename 
   ORDER BY column1 ASC/DESC;
   ```

- **Группировка**: Используется оператор `GROUP BY`.
   ```sql
   SELECT column1, SUM(column2) 
   FROM tablename 
   GROUP BY column1;
   ```

<warning>
    <p>
        В <code>SELECT</code> должны быть либо агрегатные функции, либо поля из <code>GROUP BY</code>.
    </p>
</warning>

## Транзакции и индексы

<p>
    <tooltip term="Транзакция">Транзакция</tooltip>
    объединяет несколько операций в одно целое: либо все успешны, либо ни одна. Это сохраняет
    целостность данных.
</p>

```sql
BEGIN;
-- операции
COMMIT;
```

<p>
    <tooltip term="Индекс">Индекс</tooltip>
    ускоряет поиск и сортировку по столбцам.
</p>

   ```sql
   CREATE INDEX indexname
    ON tablename (columnname);
   ```

<warning>
    <p>
        Индексы ускоряют чтение, но замедляют запись (INSERT/UPDATE/DELETE). Индексируйте
        избирательно.
    </p>
</warning>

## Практические примеры

- **Создание базы данных и таблицы**:
   ```sql
   CREATE DATABASE mydatabase;
   \c mydatabase
   CREATE TABLE users (
       id SERIAL PRIMARY KEY,
       name VARCHAR(50),
       email VARCHAR(100)
   );
   ```

- **Добавление данных**:
   ```sql
   INSERT INTO users (name, email) 
   VALUES ('John Doe', 'john@example.com');
   ```

- **Выборка данных**:
   ```sql
   SELECT * FROM users WHERE name = 'John Doe';
   ```

<note><p><code>SERIAL</code> — удобный способ создать автоинкрементный идентификатор; под капотом
    используется последовательность.</p>
</note>
- 
## Подзапросы и соединения

### Подзапросы

<p>
    <tooltip term="Подзапрос">Подзапрос</tooltip>
    — вложенный <code>SELECT</code>, который возвращает значение(я) для условия во внешнем запросе.
</p>

```sql
SELECT *
FROM users
WHERE salary > (SELECT AVG(salary) FROM users);
```

### Соединения

Соединения объединяют строки из двух таблиц по условию соответствия ключей.

#### INNER JOIN

Возвращает только те строки, которые имеют совпадения в обеих таблицах.

```sql
SELECT *
FROM users
         INNER JOIN orders
                    ON users.id = orders.user_id;
```

#### LEFT JOIN

Возвращает все строки из левой таблицы и соответствующие строки из правой таблицы. Если нет совпадений, возвращает NULL.

```sql
SELECT *
FROM users
         LEFT JOIN orders
                   ON users.id = orders.user_id;
```

#### RIGHT JOIN

Аналогично LEFT JOIN, но возвращает все строки из правой таблицы.

```sql
SELECT *
FROM users
         RIGHT JOIN orders
                    ON users.id = orders.user_id;
```

#### FULL OUTER JOIN

Возвращает все строки из обеих таблиц, заполняя NULL там, где нет совпадений.

```sql
SELECT *
FROM users
         FULL OUTER JOIN orders
                         ON users.id = orders.user_id;
```

<tip>
    <p>Используйте <code>USING (column)</code>, если имя соединяющего столбца совпадает в обеих
        таблицах.</p>
</tip>

## Агрегатные функции

Агрегатные функции используются для вычисления значений на основе набора данных.

- **SUM**: Сумма значений.
- **AVG**: Среднее значение.
- **MAX**: Максимальное значение.
- **MIN**: Минимальное значение.
- **COUNT**: Количество строк.

```sql
SELECT SUM(salary) AS total_salary
FROM users;
```

## Индексирование и оптимизация запросов

Индексирование может существенно повысить скорость выполнения запросов.

### Создание индекса

```sql
CREATE INDEX idx_name
    ON users (name);
```

### Оптимизация запросов

- Используйте `EXPLAIN` для анализа плана выполнения запроса.
- Избегайте использования `SELECT *`, если не нужно все поля.
- Используйте индексы для полей, участвующих в фильтрации и сортировке.

```sql
EXPLAIN
SELECT *
FROM users
WHERE name = 'John Doe';
```

## Безопасность и права доступа

### Создание пользователей и назначение прав

<p>В PostgreSQL пользователи — это
    <tooltip term="Роль">роли</tooltip>
    . Привилегии выдаются на объекты (таблицы, схемы, БД).
</p>

```sql
CREATE ROLE myuser WITH PASSWORD 'mypassword';
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE users TO myuser;
```

<warning><p><code>SUPERUSER</code> даёт полный доступ ко всем объектам. Используйте минимум прав по
                    принципу наименьших привилегий.</p>
</warning>

### Роли и привилегии

- **SUPERUSER**: Полный доступ к базе данных.
- **CREATEDB**: Право создавать базы данных.
- **CREATEROLE**: Право создавать роли.

```sql
ALTER ROLE myuser WITH SUPERUSER;
```

## Резервное копирование и восстановление

### Создание резервной копии

```sql
pg_dump -U myuser mydatabase > backup.sql
```

### Восстановление из резервной копии

```sql
psql -U myuser mydatabase < backup.sql
```

<note><p>Проверяйте совместимость версий <code>pg_dump</code> и сервера, а также кодировку базы.</p></note>

## Триггеры и функции

<p>
    <tooltip term="Триггер">Триггер</tooltip>
    запускает процедуру при событиях <code>INSERT</code>/<code>UPDATE</code>/<code>DELETE</code>.
</p>

```sql
CREATE OR REPLACE FUNCTION update_timestamp()
    RETURNS TRIGGER AS
$$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_timestamp_trigger
    BEFORE UPDATE
    ON users
    FOR EACH ROW
EXECUTE PROCEDURE update_timestamp();
```

Функции позволяют группировать повторяющиеся операции и использовать их в запросах.

```sql
CREATE OR REPLACE FUNCTION get_user_name(p_id INTEGER)
    RETURNS VARCHAR AS
$$
DECLARE
    v_name VARCHAR;
BEGIN
    SELECT name
    INTO v_name
    FROM users
    WHERE id = p_id;
    RETURN v_name;
END;
$$ LANGUAGE plpgsql;

SELECT get_user_name(1);
```

## Последовательности и идентификаторы

### Последовательности

Последовательности используются для создания уникальных идентификаторов.

```sql
CREATE SEQUENCE user_id_seq;

CREATE TABLE users
(
    id   INTEGER DEFAULT nextval('user_id_seq'),
    name VARCHAR(50)
);
```

### Идентификаторы (SERIAL)

SERIAL — это псевдоним для последовательности, который автоматически создается при создании таблицы.

```sql
CREATE TABLE users
(
    id   SERIAL PRIMARY KEY,
    name VARCHAR(50)
);
```

## Вью и материализованные вью

### Вью

Вью — это виртуальные таблицы, которые основаны на запросе. Они не хранят данные физически,
а вместо этого вычисляют результаты запроса каждый раз, когда к ним обращаются.

```sql
CREATE VIEW user_info AS
SELECT id, name, email
FROM users;

SELECT *
FROM user_info;
```

### Материализованные вью

Материализованные вью — это физические таблицы, которые периодически обновляются на основе запроса.
Они хранят результаты запроса в физической таблице, что может ускорить выполнение запросов.

```sql
CREATE MATERIALIZED VIEW user_info AS
SELECT id, name, email
FROM users;

REFRESH MATERIALIZED VIEW user_info;
```

### Пример материализованного представления

#### 1. Создание исходных таблиц

```sql
-- Таблица продаж
CREATE TABLE sales
(
    id         SERIAL PRIMARY KEY,
    product_id INT,
    sale_date  DATE,
    quantity   INT,
    price      DECIMAL(10, 2)
);

-- Таблица продуктов
CREATE TABLE products
(
    id       SERIAL PRIMARY KEY,
    name     VARCHAR(100),
    category VARCHAR(50)
);

-- Наполнение тестовыми данными
INSERT INTO products (name, category)
VALUES ('Ноутбук', 'Электроника'),
       ('Смартфон', 'Электроника'),
       ('Книга', 'Литература');

INSERT INTO sales (product_id, sale_date, quantity, price)
VALUES (1, '2025-01-01', 5, 100000),
       (2, '2025-01-02', 10, 50000),
       (3, '2025-01-03', 20, 1000);
```

#### 2. Создание материализованного представления

```sql
CREATE MATERIALIZED VIEW sales_summary AS
SELECT p.category,
       COUNT(s.id)               AS total_sales,
       SUM(s.quantity * s.price) AS total_revenue,
       MAX(s.sale_date)          AS last_sale_date
FROM sales s
         JOIN products p ON s.product_id = p.id
GROUP BY p.category
WITH DATA;
```

#### 3. Запрос к материализованному представлению

```sql
SELECT *
FROM sales_summary;
```

_________

| category    | total_sales | total_revenue | last_sale_date |
|-------------|-------------|---------------|----------------|
| Электроника | 2           | 1000000       | 2025-01-02     |
| Литература  | 1           | 20000         | 2025-01-03     |

### Сравнение с обычным запросом

#### Обычный агрегирующий запрос:

```sql
EXPLAIN ANALYZE
SELECT p.category,
       COUNT(s.id),
       SUM(s.quantity * s.price),
       MAX(s.sale_date)
FROM sales s
         JOIN products p ON s.product_id = p.id
GROUP BY p.category;
```

**План выполнения:**

```
GroupAggregate  (cost=30.76..30.79 rows=2 width=68)
Planning Time: 0.153 ms
Execution Time: 0.045 ms
```

#### Запрос к материализованному представлению:

```sql
EXPLAIN ANALYZE
SELECT *
FROM sales_summary;
```

**План выполнения:**

```
Seq Scan on sales_summary  (cost=0.00..1.03 rows=3 width=68)
Planning Time: 0.014 ms
Execution Time: 0.012 ms
```

### Ключевые преимущества:

1. **Производительность**
   Время выполнения сократилось с **0.045 ms** до **0.012 ms** (в 3.75 раза быстрее). Для больших данных разница будет
   более существенной.

2. **Снижение нагрузки на БД**
   Материализованное представление:
    - Не требует выполнения JOIN операций
    - Избегает повторных вычислений агрегатных функций
    - Минимизирует блокировки таблиц

3. **Возможность индексирования**

```sql
CREATE INDEX idx_sales_summary_category ON sales_summary (category);
```

Индексы на материализованных представлениях работают как на обычных таблицах.

4. **Оптимизация сложных запросов**  
   Особенно эффективны для:
    - Многотабличных JOIN-ов
    - Вложенных подзапросов
    - Рекурсивных запросов
    - Геопространственных вычислений

### Ограничения и особенности:

1. **Обновление данных**
   Требует явного обновления:

```sql
REFRESH MATERIALIZED VIEW sales_summary; -- Полное обновление
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary; -- Без блокировок (PostgreSQL 9.4+)
```

2. **Хранение данных**
   Занимает место на диске (как обычная таблица). Размер можно проверить:

```sql
SELECT pg_size_pretty(pg_total_relation_size('sales_summary'));
```

3. **Актуальность данных**
   Не отражает изменения в реальном времени. Частота обновления зависит от требований приложения.

### Когда использовать:

- Отчеты и аналитика, где допустима небольшая задержка данных
- Сложные дашборды с агрегацией
- Часто используемые ресурсоемкие запросы
- Системы с высокой read-нагрузкой

Материализованные представления особенно выгодны при работе с большими объемами данных (от миллионов записей), где
обычные агрегационные запросы становятся непозволительно медленными.

## Common Table Expressions (CTE)

CTE — это временные результаты, которые можно использовать в запросе.

Общие табличные выражения (CTE) позволяют сохранять результаты запроса на этапе построения запроса.
CTE объявляются с помощью ключевого слова WITH и могут быть использованы в основном запросе.

```sql
WITH user_data AS (SELECT id, name
                   FROM users)
SELECT *
FROM user_data
WHERE name = 'John Doe';
```

## Регулярные выражения

Регулярные выражения используются для поиска и замены текста.

```sql
SELECT *
FROM users
WHERE name ~ 'John.*';
```

## Окна (Window Functions)

Окна, или оконные функции, — это мощный инструмент SQL, позволяющий выполнять вычисления над набором строк, связанных с
текущей строкой. Они позволяют получить доступ к данным из соседних строк, что делает возможным выполнение таких
операций, как ранжирование, агрегация и сравнение значений между строками.

### Преимущества

- **Ранжирование и сортировка**: Окна позволяют ранжировать строки по определенным критериям.
- **Агрегация**: Выполняют агрегационные операции над набором строк.
- **Сравнение значений**: Позволяют сравнивать значения между соседними строками.

### Основные оконные функции

#### 1. `ROW_NUMBER()`

Назначает уникальный номер каждой строке в результате запроса.

```sql
SELECT id,
       name,
       ROW_NUMBER() OVER (ORDER BY id) AS row_num
FROM users;
```

#### 2. `RANK()` и `DENSE_RANK()`

Используются для ранжирования строк. `RANK()` оставляет пробелы в ранжировании, если есть равные значения, а
`DENSE_RANK()` не оставляет пробелов.

```sql
SELECT id,
       score,
       RANK() OVER (ORDER BY score DESC)       AS rank,
       DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM scores;
```

#### 3. `LAG()` и `LEAD()`

Позволяют получить доступ к предыдущей (`LAG()`) или следующей (`LEAD()`) строке.

```sql
SELECT id,
       name,
       LAG(name) OVER (ORDER BY id)  AS prev_name,
       LEAD(name) OVER (ORDER BY id) AS next_name
FROM users;
```

#### 4. `SUM()`, `AVG()`, `MAX()`, `MIN()`

Выполняют агрегационные операции над набором строк.

```sql
SELECT id,
       name,
       SUM(score) OVER (PARTITION BY category) AS total_score
FROM scores;
```

### Ключевые понятия

- **`OVER`**: Определяет набор строк, над которым выполняется оконная функция.
- **`PARTITION BY`**: Разделяет результат на группы по указанным столбцам.
- **`ORDER BY`**: Определяет порядок строк внутри каждой группы.
- **`ROWS` или `RANGE`**: Определяет границы окна (например, `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING`).

### Примеры использования

#### Ранжирование студентов по баллам

```sql
SELECT student_id,
       score,
       RANK() OVER (ORDER BY score DESC) AS rank
FROM exam_results;
```

#### Агрегация продаж по категориям

```sql
SELECT category,
       SUM(amount) OVER (PARTITION BY category) AS total_sales
FROM sales;
```

#### Сравнение значений между соседними строками

```sql
SELECT id,
       value,
       LAG(value) OVER (ORDER BY id)  AS prev_value,
       LEAD(value) OVER (ORDER BY id) AS next_value
FROM data;
```