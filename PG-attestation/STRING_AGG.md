


Функция `CROSSTAB()` в PostgreSQL — это способ создать **сводную таблицу (pivot table)**, поворачивая строки в **колонки**, аналогично Excel PivotTable. Она входит в расширение **`tablefunc`**, которое нужно сначала включить.

---

## 🔧 1. Как подключить `tablefunc`

```sql
CREATE EXTENSION IF NOT EXISTS tablefunc;
```

---

## 📊 2. Пример задачи

Допустим, у нас есть таблица продаж:

```sql
CREATE TABLE sales (
    employee TEXT,
    month TEXT,
    amount NUMERIC
);
```

### Примерные данные:

```sql
INSERT INTO sales VALUES
('Alice', 'Jan', 100),
('Alice', 'Feb', 150),
('Bob',   'Jan', 200),
('Bob',   'Feb', 250),
('Charlie', 'Jan', 300);
```

---

## 📌 3. Простой `CROSSTAB`

Мы хотим повернуть строки — чтобы каждый сотрудник был в строке, а каждый месяц — в колонке.

### 📘 Синтаксис:

```sql
SELECT *
FROM crosstab(
  $$SELECT employee, month, amount FROM sales ORDER BY 1,2$$
) AS ct(employee TEXT, jan NUMERIC, feb NUMERIC);
```

### 🟢 Результат:

| employee | jan | feb  |
| -------- | --- | ---- |
| Alice    | 100 | 150  |
| Bob      | 200 | 250  |
| Charlie  | 300 | NULL |

---

## ⚠️ Важно

* `crosstab()` **не динамическая** — нужно заранее указать список колонок (`jan`, `feb`, и т.д.).
* Отсутствующие значения (например, у Charlie нет Feb) будут **NULL**.

---

## 📌 4. `CROSSTAB` с двумя входными запросами (гибкий вариант)

Если нужно заранее задать **порядок столбцов**, используем второй параметр:

```sql
SELECT *
FROM crosstab(
  $$SELECT employee, month, amount FROM sales ORDER BY 1,2$$,
  $$SELECT DISTINCT month FROM sales ORDER BY month$$
) AS ct(employee TEXT, jan NUMERIC, feb NUMERIC);
```

---

## 🧠 Когда использовать `CROSSTAB()`:

* Построение сводных таблиц (Pivot Table).
* Генерация отчетов.
* Поворот категорий в фиксированные колонки.

---

Хочешь пример с динамическим количеством колонок (через PL/pgSQL) или с агрегацией (`SUM`, `COUNT`)?



Функция `STRING_AGG()` в PostgreSQL используется для объединения строк в одну, с заданным разделителем — очень полезна для агрегирования текстов.

---

## 📘 Пример на таблице `sales`

Допустим, у нас есть такая таблица:

```sql
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    employee TEXT,
    region TEXT,
    amount NUMERIC
);
```

### Данные:

```sql
INSERT INTO sales (employee, region, amount) VALUES
('Alice', 'East', 100),
('Alice', 'East', 200),
('Bob',   'West', 300),
('Bob',   'West', 400),
('Charlie', 'East', 500);
```

---

## 🔹 Пример 1: список сумм по каждому сотруднику

```sql
SELECT employee,
       STRING_AGG(amount::TEXT, ', ') AS all_sales
FROM sales
GROUP BY employee;
```

🟢 **Результат:**

| employee | all\_sales |
| -------- | ---------- |
| Alice    | 100, 200   |
| Bob      | 300, 400   |
| Charlie  | 500        |

---

## 🔹 Пример 2: список сотрудников по регионам

```sql
SELECT region,
       STRING_AGG(DISTINCT employee, ', ' ORDER BY employee) AS employees
FROM sales
GROUP BY region;
```

🟢 **Результат:**

| region | employees      |
| ------ | -------------- |
| East   | Alice, Charlie |
| West   | Bob            |

---

## 🔹 Пример 3: Составление HTML-списка

```sql
SELECT region,
       '<ul><li>' || STRING_AGG(employee, '</li><li>') || '</li></ul>' AS html_list
FROM sales
GROUP BY region;
```

---

## 🧠 Общий синтаксис:

```sql
STRING_AGG(expression, delimiter [ORDER BY ...])
```

* `expression` — что объединять (обычно текст).
* `delimiter` — разделитель (например, `', '`).
* `ORDER BY` — если важен порядок элементов в агрегате.

---
