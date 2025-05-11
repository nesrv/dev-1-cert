### Курсоры в PostgreSQL

**Курсор** в PostgreSQL — это объект, который позволяет **построчно обрабатывать результат запроса**. Курсоры особенно полезны, когда нужно последовательно обрабатывать большое количество строк, а не загружать их все в память сразу.

---

### 📌 Когда использовать курсоры:

* При работе с большим объёмом данных.
* Когда нужно пошагово обрабатывать записи в цикле.
* Внутри процедур и функций `PL/pgSQL`.

---

## 🔧 Синтаксис использования курсора

```plpgsql
DECLARE cursor_name CURSOR FOR select_query;
```

### 🔁 Работа с курсором:

1. **DECLARE** — объявление курсора.
2. **OPEN** — открытие курсора.
3. **FETCH** — извлечение данных.
4. **CLOSE** — закрытие курсора.

---

## ✅ Пример 1: Простой курсор

```sql
DO $$
DECLARE
    emp_record RECORD;
    emp_cursor CURSOR FOR SELECT id, name FROM employees;
BEGIN
    OPEN emp_cursor;
    LOOP
        FETCH emp_cursor INTO emp_record;
        EXIT WHEN NOT FOUND;

        RAISE NOTICE 'ID: %, Name: %', emp_record.id, emp_record.name;
    END LOOP;
    CLOSE emp_cursor;
END;
$$ LANGUAGE plpgsql;
```

### Что происходит:

* Объявляется курсор `emp_cursor`, выбирающий `id` и `name` из таблицы `employees`.
* Курсор открывается.
* Из него извлекаются строки по одной.
* Выводится информация.
* Курсор закрывается.

---

## ✅ Пример 2: Курсор с параметром

```sql
DO $$
DECLARE
    emp_record RECORD;
    emp_cursor CURSOR (dept_id INT) FOR
        SELECT id, name FROM employees WHERE department_id = dept_id;
BEGIN
    OPEN emp_cursor(2);  -- передаём значение параметра
    LOOP
        FETCH emp_cursor INTO emp_record;
        EXIT WHEN NOT FOUND;

        RAISE NOTICE 'ID: %, Name: %', emp_record.id, emp_record.name;
    END LOOP;
    CLOSE emp_cursor;
END;
$$ LANGUAGE plpgsql;
```

---

## ✅ Пример 3: Использование курсора в хранимой процедуре

```sql
CREATE OR REPLACE PROCEDURE process_employees()
LANGUAGE plpgsql
AS $$
DECLARE
    rec RECORD;
    cur CURSOR FOR SELECT id, name FROM employees;
BEGIN
    OPEN cur;
    LOOP
        FETCH cur INTO rec;
        EXIT WHEN NOT FOUND;

        -- Пример обработки: вывод
        RAISE NOTICE 'Employee: %, %', rec.id, rec.name;
    END LOOP;
    CLOSE cur;
END;
$$;
```

**Вызов:**

```sql
CALL process_employees();
```

---

## 🧹 Курсор как REF CURSOR (возвращаемый из функции)

```sql
CREATE OR REPLACE FUNCTION get_employees_cursor()
RETURNS REFCURSOR AS $$
DECLARE
    ref refcursor;
BEGIN
    OPEN ref FOR SELECT * FROM employees;
    RETURN ref;
END;
$$ LANGUAGE plpgsql;
```

**Пример использования:**

```sql
BEGIN;
SELECT get_employees_cursor();  -- Получаем имя курсора
FETCH ALL IN "<cursor_name>";   -- Извлекаем все строки
COMMIT;
```

---

## 📝 Полезные команды

| Команда   | Описание                          |
| --------- | --------------------------------- |
| `DECLARE` | Объявление курсора                |
| `OPEN`    | Открытие курсора                  |
| `FETCH`   | Получение строки                  |
| `MOVE`    | Пропуск строки                    |
| `CLOSE`   | Закрытие курсора                  |
| `FOR`     | Итерация по курсору в `FOR`-цикле |

---

## ⚠️ Важно:

* Курсоры требуют `BEGIN`/`COMMIT`, если используются вне PL/pgSQL.
* Без `CLOSE` курсор останется открытым и будет удерживать ресурсы.

---

Если нужно — покажу пример курсора с обновлением или удалением строк. Хотите такой?
