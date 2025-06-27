**Тема:** информационная система университета



## 1. Структура базы данных

### Таблицы:

* faculties — Факультеты
* departments — Кафедры
* groups — Учебные группы
* students — Студенты
* teachers — Преподаватели
* subjects — Учебные дисциплины
* semesters — Семестры
* courses — Курсы
* marks — Оценки
* attendances — Посещаемость
* classrooms — Аудитории
* schedules — Расписание
* exams — Экзамены
* logins — Лог входов/действий
* roles — Роли пользователей

---

## 2. Создание таблиц

Пример создания таблиц 'faculties' и 'departments':

```sql
CREATE TABLE faculties (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);
```
```sql
INSERT INTO faculties (name) VALUES
('Факультет информационных технологий'),
('Факультет математики');
```

```sql
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    faculty_id INT REFERENCES faculties(id)
);
```

```sql
INSERT INTO departments (name, faculty_id) VALUES
('Кафедра программной инженерии', 1),
('Кафедра вычислительной математики', 2);
```

![image](https://github.com/user-attachments/assets/d7144b2e-b185-40d7-9835-e82151ad6ff9)


Таблицы успешно создались и наполнились случайными данными

## 3. Создание представлений.




