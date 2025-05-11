Вот **простой пример курсора** в PostgreSQL — перебираем строки из таблицы `employees` и выводим имя и ID каждого сотрудника:

---

### 🔧 Таблица `employees` (примерная структура):

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT
);

INSERT INTO employees (name) VALUES 
('Иван'), ('Мария'), ('Алексей');
```

---

### ✅ Простой курсор в анонимном блоке:

```sql
DO $$
DECLARE
    emp_rec RECORD;
    emp_cursor CURSOR FOR SELECT id, name FROM employees;
BEGIN
    OPEN emp_cursor;

    LOOP
        FETCH emp_cursor INTO emp_rec;
        EXIT WHEN NOT FOUND;

        RAISE NOTICE 'Сотрудник: ID = %, Имя = %', emp_rec.id, emp_rec.name;
    END LOOP;

    CLOSE emp_cursor;
END;
$$ LANGUAGE plpgsql;
```

---

### 📌 Что делает этот код:

* Объявляет курсор `emp_cursor`, который выбирает `id` и `name` из таблицы `employees`.
* Открывает курсор.
* В цикле извлекает строки по одной.
* Выводит каждую строку с помощью `RAISE NOTICE`.
* Закрывает курсор после завершения.

---

Хочешь пример, где курсор не просто читает, а **изменяет** данные?
