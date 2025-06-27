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

## 2. Создание таблиц и их наполнение

Создания таблиц и наполнение данными:
```sql
CREATE TABLE faculties (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

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

INSERT INTO departments (name, faculty_id) VALUES
('Кафедра программной инженерии', 1),
('Кафедра вычислительной математики', 2);
```

```sql
CREATE TABLE groups (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    department_id INT REFERENCES departments(id),
    student_count INT DEFAULT 0
);

INSERT INTO groups (name, department_id) VALUES
('ИС-21', 1),
('ПМ-22', 2);
```

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    group_id INT REFERENCES groups(id)
);

INSERT INTO students (name, group_id) VALUES
('Иванов Иван', 1),
('Петров Петр', 1),
('Сидорова Анна', 2);
```

```sql
CREATE TABLE teachers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    department_id INT REFERENCES departments(id)
);

INSERT INTO teachers (name, department_id) VALUES
('Смирнов Алексей', 1),
('Кузнецова Мария', 2);
```

```sql
CREATE TABLE subjects (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    department_id INT REFERENCES departments(id)
);

INSERT INTO subjects (name, department_id) VALUES
('Базы данных', 1),
('Математический анализ', 2);
```

```sql
CREATE TABLE semesters (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

INSERT INTO semesters (name) VALUES
('Осенний 2024'),
('Весенний 2025');
```

```sql
CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    subject_id INT REFERENCES subjects(id),
    teacher_id INT REFERENCES teachers(id),
    semester TEXT NOT NULL
);

INSERT INTO courses (subject_id, teacher_id, semester) VALUES
(1, 1, 'Осенний 2024'),
(2, 2, 'Весенний 2025');
```

```sql
CREATE TABLE marks (
    id SERIAL PRIMARY KEY,
    student_id INT REFERENCES students(id),
    subject_id INT REFERENCES subjects(id),
    mark INT CHECK (mark >= 0 AND mark <= 5)
);

INSERT INTO marks (student_id, subject_id, mark) VALUES
(1, 1, 5),
(2, 1, 4),
(3, 2, 5);
```

```sql
CREATE TABLE attendances (
    id SERIAL PRIMARY KEY,
    student_id INT REFERENCES students(id),
    course_id INT REFERENCES courses(id),
    date DATE NOT NULL,
    status TEXT CHECK (status IN ('присутствовал', 'отсутствовал'))
);

INSERT INTO attendances (student_id, course_id, date, status) VALUES
(1, 1, '2025-03-10', 'присутствовал'),
(2, 1, '2025-03-10', 'отсутствовал'),
(3, 2, '2025-03-11', 'присутствовал');
```

```sql
CREATE TABLE classrooms (
    id SERIAL PRIMARY KEY,
    room_number TEXT NOT NULL,
    capacity INT
);

INSERT INTO classrooms (room_number, capacity) VALUES
('Ауд. 101', 30),
('Ауд. 202', 25);
```

```sql
CREATE TABLE schedules (
    id SERIAL PRIMARY KEY,
    course_id INT REFERENCES courses(id),
    teacher_id INT REFERENCES teachers(id),
    classroom_id INT REFERENCES classrooms(id),
    date_time TIMESTAMP NOT NULL
);

INSERT INTO schedules (course_id, teacher_id, classroom_id, date_time) VALUES
(1, 1, 1, '2025-03-10 09:00:00'),
(2, 2, 2, '2025-03-11 11:00:00');
```

```sql
CREATE TABLE exams (
    id SERIAL PRIMARY KEY,
    subject_id INT REFERENCES subjects(id),
    date DATE NOT NULL,
    group_id INT REFERENCES groups(id)
);

INSERT INTO exams (subject_id, date, group_id) VALUES
(1, '2025-06-01', 1),
(2, '2025-06-05', 2);
```

```sql
CREATE TABLE logins (
    id SERIAL PRIMARY KEY,
    user_id INT,
    action TEXT NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO logins (user_id, action) VALUES
(1, 'LOGIN'),
(2, 'LOGOUT');
```

```sql
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

INSERT INTO roles (name) VALUES
('Студент'),
('Преподаватель'),
('Администратор');
```


![image](https://github.com/user-attachments/assets/d7144b2e-b185-40d7-9835-e82151ad6ff9)


Таблицы успешно создались и наполнились случайными данными

## 3. Создание представлений.

```sql
CREATE VIEW vw_student_marks AS
SELECT s.name AS student_name, subj.name AS subject_name, m.mark
FROM students s
JOIN marks m ON s.id = m.student_id
JOIN subjects subj ON m.subject_id = subj.id;
```

```sql
CREATE VIEW vw_teacher_courses AS
SELECT t.name AS teacher_name, subj.name AS subject_name, c.semester
FROM teachers t
JOIN courses c ON c.teacher_id = t.id
JOIN subjects subj ON c.subject_id = subj.id;
```

```sql
CREATE MATERIALIZED VIEW vw_full_schedule AS
SELECT sch.id, sub.name AS subject, t.name AS teacher, cr.room_number, sch.date_time
FROM schedules sch
JOIN courses c ON sch.course_id = c.id
JOIN subjects sub ON c.subject_id = sub.id
JOIN teachers t ON sch.teacher_id = t.id
JOIN classrooms cr ON sch.classroom_id = cr.id;
```

```sql
CREATE VIEW vw_avg_marks_per_group AS
SELECT g.name AS group_name, AVG(m.mark) AS avg_mark
FROM groups g
JOIN students s ON s.group_id = g.id
JOIN marks m ON m.student_id = s.id
GROUP BY g.name;
```

```sql
CREATE VIEW vw_attendance_status AS
SELECT s.name AS student_name, COUNT(a.id) FILTER (WHERE a.status = 'отсутствовал') AS absences
FROM attendances a
JOIN students s ON a.student_id = s.id
GROUP BY s.name;
```
представления успешно создались: ![image](https://github.com/user-attachments/assets/31c63cac-a28d-4383-9ad3-b7cac9ba2519)

## 4. Делаем 15 запросов 3 из которых с join
1. Студенты, у которых оценка выше средней по предмету

```sql
   SELECT name FROM students
WHERE id IN (
    SELECT student_id FROM marks
    WHERE mark > (
        SELECT AVG(mark) FROM marks
        WHERE subject_id = marks.subject_id
    )
```
![image](https://github.com/user-attachments/assets/3d2ecfcf-36e8-4fee-b37f-b60604b38b4d)

3. Преподаватели, читающие более одного курса
   
```sql
SELECT name FROM teachers
WHERE id IN (
    SELECT teacher_id FROM courses
    GROUP BY teacher_id
    HAVING COUNT(*) > 1
);
```
   ![image](https://github.com/user-attachments/assets/53c3d94c-88fd-4273-b706-2d23cdaf6e49)

3. Предметы, по которым есть оценки на 5

```sql
SELECT name FROM subjects
WHERE id IN (
    SELECT subject_id FROM marks WHERE mark = 5
);
```
![image](https://github.com/user-attachments/assets/748bc4a5-37bc-4693-b014-3859f9571756)

4. Группы, в которых есть хотя бы один студент

```sql
    SELECT name FROM groups
WHERE id IN (
    SELECT DISTINCT group_id FROM students
);
```
![image](https://github.com/user-attachments/assets/c4a6b401-f125-4701-bc81-107fed044478)

5. Аудитории, в которых проводились занятия

![image](https://github.com/user-attachments/assets/1094d5a8-b3c1-42b9-963d-cacaeaff39a8)

6. Студенты, не имеющие оценок
   ![image](https://github.com/user-attachments/assets/430ee113-308c-4eba-a9f4-1e1f82bf3bf9)
   
7. Предметы, которые преподаются в осеннем семестре
   ![image](https://github.com/user-attachments/assets/d6c56ee2-c741-40df-b5c9-f01abfc0e55d)
8. Группы, у которых назначены экзамены
   ![image](https://github.com/user-attachments/assets/ede0ec58-2697-451b-9321-0fd7a0237dbc)
9. Студенты, которые посещали хотя бы одно занятие   
![image](https://github.com/user-attachments/assets/6c592526-804d-4b7d-8280-d755508cc7c9)


