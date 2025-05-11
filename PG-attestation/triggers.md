Вот краткая **шпаргалка по триггерам в PostgreSQL**:

---

## 🔹 Что такое триггеры?

**Триггер** — это механизм, который автоматически выполняет указанную функцию при определённых событиях (например, `INSERT`, `UPDATE`, `DELETE`) на таблице или представлении.

---

## 🔹 Общий синтаксис

### 1. Создание функции-триггера:

```sql
CREATE OR REPLACE FUNCTION trigger_function_name()
RETURNS trigger AS $$
BEGIN
  -- Логика триггера
  RETURN NEW; -- или RETURN OLD;
END;
$$ LANGUAGE plpgsql;
```

### 2. Создание самого триггера:

```sql
CREATE TRIGGER trigger_name
AFTER | BEFORE | INSTEAD OF
INSERT | UPDATE | DELETE | TRUNCATE
ON table_name
FOR EACH ROW | STATEMENT
WHEN (условие)
EXECUTE FUNCTION trigger_function_name();
```

---

## 🔹 Контексты данных

| Контекст | Описание                                                 |
| -------- | -------------------------------------------------------- |
| `NEW`    | Новое значение строки (доступно при `INSERT`, `UPDATE`)  |
| `OLD`    | Старое значение строки (доступно при `UPDATE`, `DELETE`) |

---

## 🔹 Примеры

### ✅ Логирование изменений:

```sql
CREATE OR REPLACE FUNCTION log_update()
RETURNS trigger AS $$
BEGIN
  INSERT INTO updates_log(table_name, changed_at)
  VALUES (TG_TABLE_NAME, now());
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER log_update_trigger
AFTER UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION log_update();
```

---

### ✅ Проверка данных перед вставкой:

```sql
CREATE OR REPLACE FUNCTION check_age()
RETURNS trigger AS $$
BEGIN
  IF NEW.age < 0 THEN
    RAISE EXCEPTION 'Возраст не может быть отрицательным';
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER age_check
BEFORE INSERT OR UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION check_age();
```

---

### ✅ Автообновление поля:

```sql
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS trigger AS $$
BEGIN
  NEW.updated_at := now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_updated_at
BEFORE UPDATE ON articles
FOR EACH ROW
EXECUTE FUNCTION update_timestamp();
```

---

## 🔹 Удаление

```sql
DROP TRIGGER trigger_name ON table_name;
DROP FUNCTION trigger_function_name();
```



### Триггеры в PostgreSQL

**Триггеры** в PostgreSQL — это механизмы, которые автоматически выполняют заданный код (например, функцию) при наступлении определённого события на таблице или представлении. Триггеры могут быть настроены на выполнение до или после определённого действия (например, `INSERT`, `UPDATE`, `DELETE`), и могут быть применены к строкам или целым таблицам.

Триггеры являются мощным инструментом для автоматизации задач, таких как поддержание целостности данных, логирование изменений или выполнение вычислений.

---

### 1. **Типы триггеров**

Триггеры в PostgreSQL можно классифицировать по нескольким параметрам:

#### **1.1. Варианты срабатывания триггеров:**

Триггеры могут срабатывать:

* **До выполнения события (BEFORE)**: триггер срабатывает перед тем, как операция (например, `INSERT`, `UPDATE`, `DELETE`) будет выполнена. Это позволяет изменить данные или отменить операцию.
* **После выполнения события (AFTER)**: триггер срабатывает после выполнения операции, когда изменения уже произведены в базе данных.
* **При условии, что операция произойдёт (INSTEAD OF)**: триггер срабатывает вместо операции, которая должна была быть выполнена. Обычно используется с представлениями.

Пример:

* **BEFORE INSERT** — выполняется перед вставкой строки в таблицу.
* **AFTER UPDATE** — выполняется после обновления строки в таблице.

#### **1.2. Варианты срабатывания на уровне строк или таблиц:**

* **FOR EACH ROW** — триггер срабатывает для каждой строки, которая была затронута операцией.
* **FOR EACH STATEMENT** — триггер срабатывает один раз для всей операции, независимо от количества затронутых строк.

---

### 2. **Создание триггера**

Для создания триггера в PostgreSQL используется команда `CREATE TRIGGER`. Сначала необходимо создать функцию, которая будет выполняться при срабатывании триггера.

#### Пример 1: Создание функции триггера

```sql
CREATE OR REPLACE FUNCTION log_employee_update() 
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO employee_audit (emp_id, action, action_time)
    VALUES (NEW.id, 'UPDATE', NOW());
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

Здесь:

* Функция `log_employee_update` вставляет запись в таблицу `employee_audit` каждый раз, когда обновляется запись в таблице `employees`.

#### Пример 2: Создание триггера

```sql
CREATE TRIGGER employee_update_trigger
AFTER UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION log_employee_update();
```

Здесь:

* Триггер `employee_update_trigger` срабатывает после обновления данных в таблице `employees`.
* Для каждой обновлённой строки выполняется функция `log_employee_update`.

---

### 3. **Удаление триггера**

Для удаления триггера в PostgreSQL используется команда `DROP TRIGGER`.

Пример:

```sql
DROP TRIGGER IF EXISTS employee_update_trigger ON employees;
```

Эта команда удаляет триггер `employee_update_trigger` с таблицы `employees`.

---

### 4. **Отключение и включение триггеров**

В PostgreSQL можно временно отключить триггеры для таблицы. Это полезно, например, при массовой загрузке данных, чтобы избежать срабатывания триггеров на каждую строку.

#### **Отключение триггеров**

Для отключения триггеров используется команда `DISABLE TRIGGER`. Эта команда отключает все триггеры или конкретные триггеры для указанной таблицы.

Пример:

```sql
DISABLE TRIGGER ALL ON employees;
```

* Эта команда отключает все триггеры на таблице `employees`.

Можно также отключить только определённый триггер:

```sql
DISABLE TRIGGER employee_update_trigger ON employees;
```

#### **Включение триггеров**

Для включения триггеров используется команда `ENABLE TRIGGER`.

Пример:

```sql
ENABLE TRIGGER ALL ON employees;
```

* Эта команда включает все триггеры на таблице `employees`.

Можно также включить конкретный триггер:

```sql
ENABLE TRIGGER employee_update_trigger ON employees;
```

---

### 5. **Просмотр информации о триггерах**

Для просмотра информации о триггерах в PostgreSQL можно использовать несколько системных представлений и функций.

#### **Пример 1: Просмотр всех триггеров в базе данных**

```sql
SELECT tgname, tgrelid::regclass, tgtype, tgenabled, tgargs
FROM pg_trigger
WHERE NOT tgisinternal;
```

Здесь:

* `tgname` — имя триггера.
* `tgrelid::regclass` — имя таблицы, к которой привязан триггер.
* `tgtype` — тип триггера (например, BEFORE, AFTER).
* `tgenabled` — статус триггера (например, 'O' для включённого, 'D' для отключённого).
* `tgargs` — аргументы триггера.

#### **Пример 2: Просмотр триггеров для конкретной таблицы**

```sql
SELECT tgname, tgtype, tgenabled
FROM pg_trigger
WHERE tgrelid = 'employees'::regclass;
```

Этот запрос покажет все триггеры, которые привязаны к таблице `employees`.

#### **Пример 3: Получение информации о триггерах с их функциями**

```sql
SELECT tgname, pg_get_triggerdef(oid) AS trigger_definition
FROM pg_trigger
WHERE NOT tgisinternal;
```

Этот запрос выводит имена триггеров и определения функций, которые они вызывают.

---

### 6. **Пример использования триггера для аудита**

Допустим, мы хотим отслеживать все изменения в таблице сотрудников (например, для логирования изменений в таблице `employees`). Мы можем создать триггер, который будет записывать изменения в отдельную таблицу аудита.

1. **Создание таблицы для аудита**:

```sql
CREATE TABLE employee_audit (
    audit_id SERIAL PRIMARY KEY,
    emp_id INT,
    action TEXT,
    action_time TIMESTAMP
);
```

2. **Создание функции триггера**:

```sql
CREATE OR REPLACE FUNCTION log_employee_update() 
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO employee_audit (emp_id, action, action_time)
    VALUES (NEW.id, 'UPDATE', NOW());
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

3. **Создание триггера для записи в таблицу аудита**:

```sql
CREATE TRIGGER employee_update_trigger
AFTER UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION log_employee_update();
```

Теперь, каждый раз, когда будет происходить обновление в таблице `employees`, информация об этом изменении будет записываться в таблицу `employee_audit`.

---

### Заключение

Триггеры в PostgreSQL — это мощный инструмент для автоматизации задач, обеспечения целостности данных и выполнения сложной логики. Они могут быть настроены на выполнение до или после изменений в данных, а также для строк или целых операций.

* **Типы триггеров**: `BEFORE`, `AFTER`, `INSTEAD OF` — срабатывают на различные события.
* **Отключение и включение триггеров**: с помощью команд `DISABLE` и `ENABLE`.
* **Просмотр информации о триггерах**: можно использовать системные представления, такие как `pg_trigger`.

Если вам нужно больше примеров или разъяснений, не стесняйтесь обращаться!








