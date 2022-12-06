# Нормальные формы
Первая нормальная форма сводится к трем правилам:
```sql
  Каждая ячейка таблицы может хранить только одно значение
  Все данные в одной колонке могут быть только одного типа
  Каждая запись в таблице должна однозначно отличаться от других записей (PRIMARY KEY)
```

Вторая нормальная форма включает в себя два пункта:
```sql
  Таблица должна быть в первой нормальной форме
  Все атрибуты (не ключевые) таблицы должны зависеть от первичного ключа
```
Третья нормальная форма, так же как и вторая, включает в себя два пункта:
```sql
  Таблица должна быть во второй нормальной форме
  Все колонки в таблице зависят от первичного ключа и не зависят друг от друга
```

# ACID
Atomicity — Атомарность\
Любая транзакция не может быть частично завершена — она либо выполнена, либо нет.

Consistency — Согласованность\
Завершившаяся транзакция должна сохранять согласованность базы данных. Другими словами, каждая успешная транзакция по определению фиксирует только допустимые результаты, при том, что в процессе работы транзакции данные могут оказываться не согласованными. В примере выше снятие денег с одного счёта приводит к тому, что данные рассинхронизированы, но в момент завершения транзакции этого нет. Гарантия согласованности данных не может быть полностью обеспечена только средствами базы данных (например, различные констрейнты). Поддержка этого свойства включает в себя работу со стороны программистов, которые пишут необходимый для этого код.

Isolation — Изолированность\
Во время выполнения транзакции параллельные транзакции не должны оказывать влияния на её результат. Ни одна транзакция не может увидеть изменения, сделанные другими незавершёнными транзакциями. Изолированность — требование дорогое, поэтому в реальных БД существуют режимы, не полностью изолирующие транзакцию (уровни изолированности Repeatable Read и ниже).

Durability — Устойчивость\
Независимо от проблем на нижних уровнях (к примеру, обесточивание системы или сбои в оборудовании) изменения, сделанные успешно завершённой транзакцией, должны остаться сохранёнными после возвращения системы в работу. Другими словами, если пользователь получил подтверждение от системы, что транзакция выполнена, он может быть уверен, что сделанные им изменения не будут отменены из-за какого-либо сбоя.

# Команды
```sql
\l - список всех баз данных
\dt - список всех таблиц
\d courses - просмотр структуры таблицы
```
# DDL (DATA DEFINITION LANGUAGE)
Создание таблицы
```sql
CREATE TABLE courses (
    -- Одновременное использование и первичного ключа и автогенерации
    id            bigint PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    name          varchar(255) UNIQUE,
    lessons_count integer NOT NULL,
    body          text,
    -- Внешний ключ
    user_id bigint REFERENCES users (id)
    product_id bigint REFERENCES products (id) ON DELETE SET NULL
);

integer 4 байта типичный выбор для целых чисел
bigint 	8 байт целое в большом диапазоне
timestamp 8 байт 	дата и время (без часового пояса)
date 	4 байта 	дата (без времени суток) 
time 	8 байт 	время суток (без даты)
boolean 1 байт TRUE 't' 'true' 'y' 'yes' 'on' '1'
NULL 
character varying(n), varchar(n) строка ограниченной переменной длины
text 	строка неограниченной переменной длины
```
Удаление таблицы
```sql
DROP TABLE courses;
```
Добавление колонки
```sql
ALTER TABLE users ADD COLUMN age integer;
```
Переименование колонки
```sql
ALTER TABLE courses RENAME COLUMN example1 TO example2;
```
Удаление колонки
```sql
ALTER TABLE courses DROP COLUMN example2;
```
Обновление колонки
```sql
ALTER TABLE addresses ADD PRIMARY KEY (id);
ALTER TABLE addresses ALTER COLUMN created_at SET DATA TYPE timestamp, ALTER COLUMN street DROP NOT NULL;
ALTER TABLE products ADD UNIQUE (product_id);
```

# DML (DATA MANIPULATION LANGUAGE)
Вставка (добавление) данных в таблицу
```sql
INSERT INTO courses (name, slug, lessons_count, body) VALUES ('basics of programming', 'basics', 10, 'this is theory');
INSERT INTO courses (name, slug) VALUES ('Bash', 'bash'), ('PHP', 'php'), ('Ruby', 'ruby');
INSERT INTO courses VALUES ('linux', 'linux', 3, 'something about linux');
```
Обновление данных
```sql
UPDATE courses SET body = 'updated!', name = 'Bash' WHERE slug = 'bash';
UPDATE courses SET name = 'another new name' WHERE (lessons_count < 2 AND lessons_count > 8) OR slug = 'linux';
UPDATE courses SET body = 'oops';
```
Удаление данных
```sql
DELETE FROM courses WHERE slug = 'bash';
```
Полное очищение таблицы
```sql
TRUNCATE courses;
```
# DQL (Data Query Language)
Выборка данных
```sql
-- Все поля из users
SELECT * FROM users;
-- Конкретные поля с псеводнимом AS
SELECT username AS name FROM users;
-- Строгий порядок: WHERE, ORDER BY, DESC(ASK), LIMIT
SELECT username, created_at FROM users WHERE birthday < '2018-10-21' ORDER BY birthday DESC LIMIT 2;

-- ORDER BY
SELECT * FROM users ORDER BY created_at;
SELECT * FROM users ORDER BY created_at ASC;
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users ORDER BY first_name, created_at;
SELECT * FROM users ORDER BY first_name DESC, created_at DESC;
SELECT first_name, created_at FROM users ORDER BY first_name ASC, created_at DESC;

-- поля, содержащие NULL, идут первыми (последними)
SELECT * FROM users ORDER BY created_at ASC NULLS FIRST;
SELECT * FROM users ORDER BY created_at DESC NULLS LAST;

-- WHERE
---- Идентификатор
SELECT * FROM users WHERE id = 3;
SELECT * FROM users WHERE id != 3;
---- NULL
SELECT * FROM users WHERE first_name IS NULL;
SELECT * FROM users WHERE created_at IS NOT NULL;
---- Строки
SELECT * FROM users WHERE first_name = 'sunny';
---- ДАТЫ
SELECT * FROM users WHERE created_at < '2018-10-05';
---- Логические операторы (AND, OR)
SELECT * FROM users WHERE first_name = 'Sunny' OR (created_at > '2018-01-01' AND created_at < '2018-10-05');
---- Диапазоны (BETWEEN)
SELECT * FROM users WHERE created_at BETWEEN '2018-01-01' AND '2018-10-05';
---- IN
SELECT * FROM users WHERE id IN (1, 2, 5);
---- LIKE, NOT LIKE (ILIKE - без регистра)
SELECT * FROM users WHERE first_name LIKE 'A%';
---- ~, !~ (оператор с регулярными выражениями POSIX)
SELECT * FROM users WHERE first_name ~ '^(A|Boe)';
---- LIMIT
SELECT * FROM users LIMIT 10;
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 30;

-- Уникальность (DISTINCT)
SELECT DISTINCT first_name FROM users;
SELECT DISTINCT first_name, last_name FROM users; -- имена и фамилии повторяться могут, но пара всегда уникальна
SELECT COUNT(DISTINCT first_name) FROM users;
-- DISTINCT ON позволяет отдельно указывать поля, по которым проверяется уникальность, и поля, которые должны оказаться в результирующей выборке.
SELECT DISTINCT ON (user_id, title) user_id, title FROM topics;
SELECT DISTINCT ON (user_id) id, user_id, title, created_at FROM topics ORDER BY user_id, created_at;

-- Aгрегатные функции
-- Когда аргументом функции является *, она считает количество строк.
-- Если в неё передать имя конкретного поля, то она посчитает количество строк, в которых это поле НЕ равно NULL
---- COUNT
SELECT COUNT(*) FROM users;
---- Max, Min
SELECT MAX(birthday) FROM users WHERE gender = 'male';
SELECT MIN(birthday) FROM users WHERE gender = 'female';
---- Sum
SELECT SUM(amount) FROM orders WHERE created_at BETWEEN '2016-01-01' AND '2016-12-31';
---- AVG
SELECT AVG(amount) FROM orders WHERE created_at BETWEEN '2016-01-01' AND '2016-12-31';

-- GROUP BY
-- Группирует строки по определённому признаку для выполнения подсчётов внутри каждой группы независимо от других групп.
SELECT user_id, COUNT(*) FROM topics GROUP BY user_id;
SELECT user_id, COUNT(*) FROM topics GROUP BY user_id ORDER BY count DESC LIMIT 3;

-- HAVING
-- Анализ только по некоторым группам. 
SELECT user_id, COUNT(*) FROM topics GROUP BY user_id HAVING COUNT(*) > 1;

-- INNER JOIN
-- В эту выборку попадают только те записи, для которых есть соответствие в другой таблице.
SELECT first_name, title FROM users JOIN topics ON users.id = topics.user_id LIMIT 5;

-- LEFT JOIN берёт все данные из одной таблицы и присоединяет к ним данные из другой, если они присутствуют. 
-- Если нет, то заполняет их NULL. Чисто технически этот запрос отличается только тем, что добавляется слово LEFT:
SELECT first_name, title FROM users LEFT JOIN topics ON users.id = topics.user_id LIMIT 5;

-- Подзапросы
-- Подзапросы могут присутствовать в предложениях SELECT, FROM, WHERE и HAVING, WITH
SELECT count( * ) FROM bookings WHERE total_amount > ( SELECT avg( total_amount ) FROM bookings );
-- IN
SELECT departure_city FROM routes WHERE departure_city IN (SELECT city FROM airports WHERE timezone ~ 'Krasnoyarsk')
-- EXISTS (NOT EXISTS)
SELECT DISTINCT a.city FROM airports a WHERE NOT EXISTS ( SELECT * FROM routes r WHERE r.departure_city = 'Москва' AND r.arrival_city = a.city)
```
# Транзакционность
```sql
BEGIN;
SELECT amount FROM accounts WHERE user_id = 10;
UPDATE accounts SET amount = amount - 50 WHERE user_id = 10;
UPDATE accounts SET amount = amount + 50 WHERE user_id = 30;
COMMIT;
```

# Производительность
Индексы
```sql
-- Создание индекса
CREATE INDEX user_birthday ON users(birthday)
-- Удаление индекса
DROP INDEX user_birthday;
-- Создание уникального индекса
CREATE UNIQUE INDEX aircrafts_unique_model_key ON aircrafts (model);
-- Индексы на основе выражений
CREATE UNIQUE INDEX aircrafts_unique_model_key ON aircrafts ( lower( model ) );
-- Частичные индексы
CREATE INDEX bookings_book_date_part_key ON bookings ( book_date ) WHERE total_amount > 1000000;
```

EXPLAIN
```sql
EXPLAIN SELECT * FROM users
  JOIN topics ON users.id = topics.user_id
  WHERE users.created_at > '10.10.2018';

                                       QUERY PLAN
-----------------------------------------------------------------------------------------
 Hash Join  (cost=10.66..23.59 rows=42 width=2377)
   Hash Cond: (topics.user_id = users.id)
   ->  Seq Scan on topics  (cost=0.00..11.30 rows=130 width=572)
   ->  Hash  (cost=10.50..10.50 rows=13 width=1805)
         ->  Seq Scan on users  (cost=0.00..10.50 rows=13 width=1805)
               Filter: (created_at > '2018-10-10 00:00:00'::timestamp without time zone)
(6 rows)

```
