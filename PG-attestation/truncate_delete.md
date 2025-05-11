Отличный вопрос — `TRUNCATE` и `DELETE` в PostgreSQL **оба удаляют данные из таблицы**, но **делают это по-разному**.

---

## 🧨 Главное отличие:

|                       | `DELETE`                         | `TRUNCATE`                      |
| --------------------- | -------------------------------- | ------------------------------- |
| **Скорость**          | Медленно (строка за строкой)     | Очень быстро (обрезает таблицу) |
| **Возврат**           | Можно `ROLLBACK` (да)            | Можно `ROLLBACK` (да)           |
| **Триггеры**          | **Срабатывают** (`AFTER DELETE`) | **Не срабатывают**              |
| **WHERE**             | Да — можно выборочно             | ❌ — удаляет **всё**             |
| **Сброс ID**          | Нет                              | Да (если есть `SERIAL`)         |
| **Блокировки**        | Блокирует строки                 | Блокирует **всю таблицу**       |
| **Влияние на VACUUM** | Требует `VACUUM` после           | Не требует                      |

---

## 🔍 Примеры:

### `DELETE`: удаляет **по условию** и вызывает триггеры

```sql
DELETE FROM sales WHERE employee = 'Alice';
```

* Можно откатить.
* Триггеры `AFTER DELETE` сработают.
* Можно удалить часть данных.

---

### `TRUNCATE`: удаляет **всё** быстро

```sql
TRUNCATE TABLE sales;
```

* Нельзя использовать `WHERE`.
* Очень быстро — очищает таблицу без чтения строк.
* Сбросит счётчик `SERIAL` (если не указать `RESTART IDENTITY OFF`).

---

## ⚠️ Влияние на `SERIAL` / `IDENTITY`

```sql
TRUNCATE TABLE sales RESTART IDENTITY;
-- ID снова начнется с 1
```

```sql
TRUNCATE TABLE sales CONTINUE IDENTITY;
-- ID продолжит с текущего значения
```

---

## ✅ Когда использовать:

| Ситуация                        | Используй                    |
| ------------------------------- | ---------------------------- |
| Удалить выборочно               | `DELETE`                     |
| Нужно вызвать триггеры          | `DELETE`                     |
| Очистить **всю** таблицу быстро | `TRUNCATE`                   |
| Освободить место, сбросить ID   | `TRUNCATE`                   |
| Есть внешние ключи — осторожно! | Лучше `DELETE` или `CASCADE` |

---

Хочешь пример, где `TRUNCATE` не сработает из-за внешнего ключа (`FOREIGN KEY`)?





В PostgreSQL можно управлять **целостностью данных** через ограничения (constraints), такие как:

1. **Внешние ключи (`FOREIGN KEY`)**
2. **Значения по умолчанию (`DEFAULT`)**
3. **Уникальность (`UNIQUE`)**

Вот как добавлять эти ограничения — при создании таблицы и **на уже существующую таблицу**.

---

## 📌 1. **Внешний ключ (`FOREIGN KEY`)**

Обеспечивает ссылочную целостность: значение должно существовать в другой таблице.

### При создании таблицы:

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id)
);
```

Или явно:

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

### К существующей таблице:

```sql
ALTER TABLE orders
ADD CONSTRAINT fk_customer
FOREIGN KEY (customer_id) REFERENCES customers(id);
```

---

## 📌 2. **Значение по умолчанию (`DEFAULT`)**

Автоматически присваивает значение, если оно не указано при вставке.

### При создании таблицы:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    price NUMERIC DEFAULT 0.0,
    created_at TIMESTAMP DEFAULT now()
);
```

### К существующей таблице:

```sql
ALTER TABLE products
ALTER COLUMN price SET DEFAULT 0.0;
```

Удалить `DEFAULT`:

```sql
ALTER TABLE products
ALTER COLUMN price DROP DEFAULT;
```

---

## 📌 3. **Уникальное ограничение (`UNIQUE`)**

Гарантирует, что значения в колонке (или наборе колонок) уникальны.

### При создании таблицы:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT UNIQUE
);
```

Или с явным именем:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT,
    CONSTRAINT unique_email UNIQUE (email)
);
```

### К существующей таблице:

```sql
ALTER TABLE users
ADD CONSTRAINT unique_email UNIQUE (email);
```

---

## 🔐 Комбинированные ограничения

Можно добавить `UNIQUE` на несколько колонок:

```sql
ALTER TABLE orders
ADD CONSTRAINT unique_order_combo UNIQUE (customer_id, product_id);
```

---

## 💡 Проверка ограничения (CHECK)

Дополнительно можно задавать ограничения, например:

```sql
ALTER TABLE products
ADD CONSTRAINT check_price_positive CHECK (price > 0);
```

---