
## 🔹 1. `GROUPING SETS` — Явное указание группировок

Позволяет задавать **наборы группировок**, которые нужно получить.

### Пример:

```sql
SELECT region, employee, SUM(amount)
FROM sales
GROUP BY GROUPING SETS (
  (region, employee),  -- по региону и сотруднику
  (region),            -- только по региону
  ()                   -- общая сумма (всего)
);
```

🟢 Аналог `UNION ALL` трех отдельных `GROUP BY`.

---

## 🔹 2. `ROLLUP` — Иерархическая агрегация

Создает **агрегации по уровням иерархии**: от более детальных к обобщенным.

### Пример:

```sql
SELECT region, employee, SUM(amount)
FROM sales
GROUP BY ROLLUP(region, employee);
```

🔹 Возвращает:

* Сумму по каждому сотруднику в регионе.
* Сумму по региону.
* Общую сумму.

📌 Это эквивалент:

```sql
GROUPING SETS (
  (region, employee),
  (region),
  ()
)
```

---

## 🔹 3. `CUBE` — Полная многомерная агрегация

Вычисляет **все возможные комбинации** группировок по указанным полям.

### Пример:

```sql
SELECT region, employee, SUM(amount)
FROM sales
GROUP BY CUBE(region, employee);
```

🔹 Возвращает:

* Сумму по каждой паре (region, employee),
* Сумму по каждому `region` (все сотрудники),
* Сумму по каждому `employee` (все регионы),
* Общую сумму (все вместе).

📌 Эквивалент:

```sql
GROUPING SETS (
  (region, employee),
  (region),
  (employee),
  ()
)
```

---

## 🔹 Как понять, какие строки обобщенные?

Используйте функцию `GROUPING(...)` — она возвращает `1`, если поле **свернуто (NULL из-за агрегации)**, и `0` если оно реально из данных.

### Пример с `ROLLUP`:

```sql
SELECT
  region,
  employee,
  SUM(amount),
  GROUPING(region) AS is_region_total,
  GROUPING(employee) AS is_employee_total
FROM sales
GROUP BY ROLLUP(region, employee);
```

---

## 🧠 Сравнение:

| Конструкция     | Что делает                                       |
| --------------- | ------------------------------------------------ |
| `GROUPING SETS` | Явно указываешь комбинации группировок           |
| `ROLLUP`        | Делает вложенные уровни иерархии (слева направо) |
| `CUBE`          | Все возможные комбинации                         |

---
