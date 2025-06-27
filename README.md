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
-- Создание и наполнение базы данных университетской системы одним запросом (исправленная версия)

-- 1. Создание таблиц

-- Факультеты
CREATE TABLE faculties (
    faculty_id SERIAL PRIMARY KEY,
    faculty_name VARCHAR(100) NOT NULL
);

-- Кафедры
CREATE TABLE departments (
    department_id SERIAL PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL,
    faculty_id INTEGER REFERENCES faculties(faculty_id)
);

-- Учебные группы
CREATE TABLE groups (
    group_id SERIAL PRIMARY KEY,
    group_name VARCHAR(50) NOT NULL,
    department_id INTEGER REFERENCES departments(department_id)
);

-- Студенты
CREATE TABLE students (
    student_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    group_id INTEGER REFERENCES groups(group_id)
);

-- Преподаватели
CREATE TABLE teachers (
    teacher_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    department_id INTEGER REFERENCES departments(department_id)
);

-- Учебные дисциплины
CREATE TABLE subjects (
    subject_id SERIAL PRIMARY KEY,
    subject_name VARCHAR(100) NOT NULL,
    department_id INTEGER REFERENCES departments(department_id)
);

-- Семестры
CREATE TABLE semesters (
    semester_id SERIAL PRIMARY KEY,
    semester_number INTEGER NOT NULL,
    academic_year VARCHAR(9) NOT NULL
);

-- Курсы
CREATE TABLE courses (
    course_id SERIAL PRIMARY KEY,
    subject_id INTEGER REFERENCES subjects(subject_id),
    teacher_id INTEGER REFERENCES teachers(teacher_id),
    semester_id INTEGER REFERENCES semesters(semester_id)
);

-- Оценки
CREATE TABLE marks (
    mark_id SERIAL PRIMARY KEY,
    student_id INTEGER REFERENCES students(student_id),
    course_id INTEGER REFERENCES courses(course_id),
    mark INTEGER CHECK (mark >= 0 AND mark <= 100),
    date DATE NOT NULL
);

-- Посещаемость
CREATE TABLE attendances (
    attendance_id SERIAL PRIMARY KEY,
    student_id INTEGER REFERENCES students(student_id),
    course_id INTEGER REFERENCES courses(course_id),
    date DATE NOT NULL,
    status VARCHAR(20) CHECK (status IN ('присутствовал', 'отсутствовал', 'опоздал'))
);

-- Аудитории
CREATE TABLE classrooms (
    classroom_id SERIAL PRIMARY KEY,
    room_number VARCHAR(20) NOT NULL,
    building VARCHAR(50) NOT NULL,
    capacity INTEGER NOT NULL
);

-- Расписание (исправлено: day_of_week увеличен до VARCHAR(11))
CREATE TABLE schedules (
    schedule_id SERIAL PRIMARY KEY,
    course_id INTEGER REFERENCES courses(course_id),
    classroom_id INTEGER REFERENCES classrooms(classroom_id),
    day_of_week VARCHAR(11) NOT NULL,
    start_time TIME NOT NULL,
    end_time TIME NOT NULL
);

-- Экзамены
CREATE TABLE exams (
    exam_id SERIAL PRIMARY KEY,
    course_id INTEGER REFERENCES courses(course_id),
    classroom_id INTEGER REFERENCES classrooms(classroom_id),
    exam_date DATE NOT NULL,
    start_time TIME NOT NULL
);

-- Роли пользователей
CREATE TABLE roles (
    role_id SERIAL PRIMARY KEY,
    role_name VARCHAR(50) NOT NULL
);

-- Лог входов/действий
CREATE TABLE logins (
    login_id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    role_id INTEGER REFERENCES roles(role_id),
    login_time TIMESTAMP NOT NULL,
    action VARCHAR(100) NOT NULL
);

-- 2. Наполнение таблиц тестовыми данными

INSERT INTO faculties (faculty_name) VALUES
    ('Факультет информатики'),
    ('Факультет математики'),
    ('Факультет физики');

INSERT INTO departments (department_name, faculty_id) VALUES
    ('Программная инженерия', 1),
    ('Наука о данных', 1),
    ('Прикладная математика', 2);

INSERT INTO groups (group_name, department_id) VALUES
    ('ПИ-101', 1),
    ('НД-201', 2),
    ('ПМ-301', 3);

INSERT INTO students (first_name, last_name, group_id) VALUES
    ('Иван', 'Иванов', 1),
    ('Мария', 'Петрова', 2),
    ('Алексей', 'Сидоров', 3);

INSERT INTO teachers (first_name, last_name, department_id) VALUES
    ('Сергей', 'Козлов', 1),
    ('Елена', 'Смирнова', 2),
    ('Дмитрий', 'Попов', 3);

INSERT INTO subjects (subject_name, department_id) VALUES
    ('Программирование', 1),
    ('Машинное обучение', 2),
    ('Математический анализ', 3);

INSERT INTO semesters (semester_number, academic_year) VALUES
    (1, '2024-2025'),
    (2, '2024-2025');

INSERT INTO courses (subject_id, teacher_id, semester_id) VALUES
    (1, 1, 1),
    (2, 2, 1),
    (3, 3, 2);

INSERT INTO marks (student_id, course_id, mark, date) VALUES
    (1, 1, 85, '2024-11-01'),
    (2, 2, 90, '2024-11-02'),
    (3, 3, 78, '2024-11-03');

INSERT INTO attendances (student_id, course_id, date, status) VALUES
    (1, 1, '2024-11-01', 'присутствовал'),
    (2, 2, '2024-11-02', 'отсутствовал'),
    (3, 3, '2024-11-03', 'опоздал');

INSERT INTO classrooms (room_number, building, capacity) VALUES
    ('А101', 'Главный корпус', 30),
    ('Б202', 'Научный блок', 40),
    ('В303', 'Математический корпус', 25);

INSERT INTO schedules (course_id, classroom_id, day_of_week, start_time, end_time) VALUES
    (1, 1, 'Понедельник', '09:00', '10:30'),
    (2, 2, 'Вторник', '10:00', '11:30'),
    (3, 3, 'Среда', '11:00', '12:30');

INSERT INTO exams (course_id, classroom_id, exam_date, start_time) VALUES
    (1, 1, '2024-12-15', '09:00'),
    (2, 2, '2024-12-16', '10:00'),
    (3, 3, '2024-12-17', '11:00');

INSERT INTO roles (role_name) VALUES
    ('Студент'),
    ('Преподаватель'),
    ('Администратор');

INSERT INTO logins (user_id, role_id, login_time, action) VALUES
    (1, 1, '2024-11-01 08:00:00', 'Вход в систему'),
    (1, 2, '2024-11-01 08:30:00', 'Ввод оценки'),
    (2, 1, '2024-11-01 09:00:00', 'Просмотр расписания');
```


![image](https://github.com/user-attachments/assets/b0c698ef-b36d-4cda-b23f-2187e58b964a)



Таблицы успешно создались и наполнились данными

## 3. Создание представлений.

Обычное представление: курсы студентов
```sql
CREATE VIEW student_courses AS
SELECT s.student_id, s.first_name, s.last_name, sub.subject_name, t.first_name AS teacher_first_name
FROM students s
JOIN groups g ON s.group_id = g.group_id
JOIN courses c ON c.semester_id IN (SELECT semester_id FROM semesters WHERE academic_year = '2024-2025')
JOIN subjects sub ON c.subject_id = sub.subject_id
JOIN teachers t ON c.teacher_id = t.teacher_id;
```

Материализованное представление: успеваемость студентов
```sql
CREATE MATERIALIZED VIEW student_performance AS
SELECT s.student_id, s.first_name, s.last_name, AVG(m.mark) as avg_mark
FROM students s
JOIN marks m ON s.student_id = m.student_id
GROUP BY s.student_id, s.first_name, s.last_name
HAVING AVG(m.mark) > 0
WITH DATA;
```

представления успешно создались: ![image](https://github.com/user-attachments/assets/d9972c05-450f-40c3-9620-b0bd271b9605)


## 4. Делаем 15 запросов 3 из которых с join
Пример запроса с join:
Оценки студентов с информацией о курсах

```sql
SELECT s.first_name, s.last_name, sub.subject_name, m.mark
FROM students s
JOIN marks m ON s.student_id = m.student_id
JOIN courses c ON m.course_id = c.course_id
JOIN subjects sub ON c.subject_id = sub.subject_id
WHERE m.date >= '2024-11-01';
```
![image](https://github.com/user-attachments/assets/d630c431-9ff7-45fa-bbd3-ed0a5dd60dc4)

остальные запросы с join:
```sql
Расписание с деталями аудиторий
SELECT sub.subject_name, c.room_number, s.day_of_week, s.start_time
FROM schedules s
JOIN courses co ON s.course_id = co.course_id
JOIN subjects sub ON co.subject_id = sub.subject_id
JOIN classrooms c ON s.classroom_id = c.classroom_id
WHERE s.day_of_week = 'Понедельник';

Расписание преподавателей
SELECT t.first_name, t.last_name, sub.subject_name, s.day_of_week, s.start_time
FROM teachers t
JOIN courses c ON t.teacher_id = c.teacher_id
JOIN subjects sub ON c.subject_id = sub.subject_id
JOIN schedules s ON c.course_id = s.course_id;
```
Пример вложенного запроса: 

Студенты с оценками выше среднего
```sql
SELECT first_name, last_name
FROM students
WHERE student_id IN (
    SELECT student_id
    FROM marks
    GROUP BY student_id
    HAVING AVG(mark) > (SELECT AVG(mark) FROM marks)
);
```
![image](https://github.com/user-attachments/assets/ada7e57b-f090-4fa3-9b19-b006bd3c8a32)
остальные вложенные запросы:

```sql
Курсы с более чем 2 студентами
SELECT subject_name
FROM subjects
WHERE subject_id IN (
    SELECT subject_id
    FROM courses
    WHERE course_id IN (
        SELECT course_id
        FROM marks
        GROUP BY course_id
        HAVING COUNT(DISTINCT student_id) > 2
    )
);

Преподаватели, преподающие в определенном здании
SELECT first_name, last_name
FROM teachers
WHERE teacher_id IN (
    SELECT teacher_id
    FROM courses
    WHERE course_id IN (
        SELECT course_id
        FROM schedules
        WHERE classroom_id IN (
            SELECT classroom_id
            FROM classrooms
            WHERE building = 'Главный корпус'
        )
    )
);

Студенты с идеальной посещаемостью
SELECT first_name, last_name
FROM students
WHERE student_id NOT IN (
    SELECT student_id
    FROM attendances
    WHERE status IN ('отсутствовал', 'опозלא')
);

Дисциплины с предстоящими экзаменами
SELECT subject_name
FROM subjects
WHERE subject_id IN (
    SELECT subject_id
    FROM courses
    WHERE course_id IN (
        SELECT course_id
        FROM exams
        WHERE exam_date > CURRENT_DATE
    )
);

Студенты определенной кафедры
SELECT first_name, last_name
FROM students
WHERE group_id IN (
    SELECT group_id
    FROM groups
    WHERE department_id IN (
        SELECT department_id
        FROM departments
        WHERE department_name = 'Программная инженерия'
    )
);

Преподаватели с несколькими курсами
SELECT first_name, last_name
FROM teachers
WHERE teacher_id IN (
    SELECT teacher_id
    FROM courses
    GROUP BY teacher_id
    HAVING COUNT(*) > 1
);

Аудитории с высокой вместимостью
SELECT room_number
FROM classrooms
WHERE capacity > (
    SELECT AVG(capacity)
    FROM classrooms
);

Студенты с недавними входами
SELECT first_name, last_name
FROM students
WHERE student_id IN (
    SELECT user_id
    FROM logins
    WHERE login_time > CURRENT_DATE - INTERVAL '1 day'
);

Дисциплины текущего семестра
SELECT subject_name
FROM subjects
WHERE subject_id IN (
    SELECT subject_id
    FROM courses
    WHERE semester_id = (
        SELECT semester_id
        FROM semesters
        WHERE academic_year = '2024-2025'
        AND semester_number = 1
    )
);

Студенты с высокими оценками
SELECT first_name, last_name
FROM students
WHERE student_id IN (
    SELECT student_id
    FROM marks
    WHERE mark > 90
);

Активные преподаватели
SELECT first_name, last_name
FROM teachers
WHERE teacher_id IN (
    SELECT teacher_id
    FROM courses
    WHERE semester_id IN (
        SELECT semester_id
        FROM semesters
        WHERE academic_year = '2024-2025'
    )
);
```
## 5. Агрегатные запросы (10 штук)
пример из COUNT

Подсчитывает количество студентов на каждой кафедре
```sql
SELECT department_name, COUNT(*) as student_count
FROM departments d
JOIN groups g ON d.department_id = g.department_id
JOIN students s ON g.group_id = s.group_id
GROUP BY department_name;
```
![image](https://github.com/user-attachments/assets/0f350dce-6aa7-4720-a3dc-18e92502a75e)

Другие COUNT запросы:

```sql
Подсчитывает количество экзаменов по каждой дисциплине
SELECT subject_name, COUNT(*) as exam_count
FROM subjects s
JOIN courses c ON s.subject_id = c.subject_id
JOIN exams e ON c.course_id = e.course_id
GROUP BY subject_name;

Подсчитывает количество аудиторий в каждом здании
SELECT building, COUNT(*) as classroom_count
FROM classrooms
GROUP BY building;

Подсчитывает количество входов в систему по ролям
SELECT role_name, COUNT(*) as login_count
FROM roles r
JOIN logins l ON r.role_id = l.role_id
GROUP BY role_name;

Подсчитывает количество студентов в каждой группе
SELECT group_name, COUNT(*) as student_count
FROM groups g
JOIN students s ON g.group_id = s.group_id
GROUP BY group_name;

Подсчитывает количество курсов, преподаваемых каждым преподавателем
SELECT teacher_id, COUNT(*) as course_count
FROM courses
GROUP BY teacher_id;

Подсчитывает количество курсов в каждом семестре
SELECT semester_number, COUNT(*) as course_count
FROM semesters s
JOIN courses c ON s.semester_id = c.semester_id
GROUP BY semester_number;

Подсчитывает количество занятий по дням недели
SELECT day_of_week, COUNT(*) as schedule_count
FROM schedules
GROUP BY day_of_week;

Подсчитывает количество записей посещаемости по статусам
SELECT status, COUNT(*) as attendance_count
FROM attendances
GROUP BY status;

Подсчитывает количество кафедр на каждом факультете
SELECT faculty_name, COUNT(*) as department_count
FROM faculties f
JOIN departments d ON f.faculty_id = d.faculty_id
GROUP BY faculty_name;
```
Пример AVG:

Вычисляет среднюю оценку для каждого студента

```sql
SELECT s.first_name, s.last_name, AVG(m.mark) as average_mark
FROM students s
JOIN marks m ON s.student_id = m.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```
![image](https://github.com/user-attachments/assets/e4993311-95af-46ab-a58b-8c83f0b754e8)

Остальные AVG:

```sql
Вычисляет среднюю оценку по каждой дисциплине
SELECT subject_name, AVG(mark) as avg_subject_mark
FROM subjects s
JOIN courses c ON s.subject_id = c.subject_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY subject_name;

Вычисляет среднюю вместимость аудиторий по кафедрам
SELECT department_name, AVG(capacity) as avg_classroom_capacity
FROM departments d
JOIN classrooms c ON c.building IN (
    SELECT building
    FROM classrooms
    WHERE department_id = d.department_id
)
GROUP BY department_name;

Вычисляет среднюю оценку по каждой группе
SELECT group_name, AVG(mark) as avg_group_mark
FROM groups g
JOIN students s ON g.group_id = s.group_id
JOIN marks m ON s.student_id = m.student_id
GROUP BY group_name;

Вычисляет среднюю оценку по курсам каждого преподавателя
SELECT teacher_id, AVG(mark) as avg_teacher_mark
FROM courses c
JOIN marks m ON c.course_id = m.course_id
GROUP BY teacher_id;

Вычисляет среднюю оценку по семестрам
SELECT semester_number, AVG(mark) as avg_semester_mark
FROM semesters s
JOIN courses c ON s.semester_id = c.semester_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY semester_number;

Вычисляет среднюю вместимость аудиторий по зданиям
SELECT building, AVG(capacity) as avg_capacity
FROM classrooms
GROUP BY building;

Вычисляет среднюю оценку по каждой дисциплине (дубликат для полноты)
SELECT subject_name, AVG(mark) as avg_subject_mark
FROM subjects s
JOIN courses c ON s.subject_id = c.subject_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY subject_name;

Вычисляет среднее количество входов в систему на пользователя по ролям
SELECT role_name, COUNT(*) / COUNT(DISTINCT user_id) as avg_logins_per_user
FROM roles r
JOIN logins l ON r.role_id = l.role_id
GROUP BY role_name;

Вычисляет среднюю оценку по факультетам
SELECT faculty_name, AVG(mark) as avg_faculty_mark
FROM faculties f
JOIN departments d ON f.faculty_id = d.faculty_id
JOIN groups g ON d.department_id = g.department_id
JOIN students s ON g.group_id = s.group_id
JOIN marks m ON s.student_id = m.student_id
GROUP BY faculty_name;
```
Пример MAX:

Находит максимальную оценку для каждого студента

```sql
SELECT s.first_name, s.last_name, MAX(m.mark) as highest_mark
FROM students s
JOIN marks m ON s.student_id = m.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```
![image](https://github.com/user-attachments/assets/55bb96a8-94d3-4f0d-9c25-dc56dca48679)
Остальные MAX:

```sql
Находит максимальную оценку по каждой дисциплине
SELECT subject_name, MAX(mark) as highest_subject_mark
FROM subjects s
JOIN courses c ON s.subject_id = c.subject_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY subject_name;

-- Находит максимальную вместимость аудиторий по кафедрам
SELECT department_name, MAX(capacity) as max_classroom_capacity
FROM departments d
JOIN classrooms c ON c.building IN (
    SELECT building
    FROM classrooms
    WHERE department_id = d.department_id
)
GROUP BY department_name;

Находит максимальную оценку по каждой группе
SELECT group_name, MAX(mark) as max_group_mark
FROM groups g
JOIN students s ON g.group_id = s.group_id
JOIN marks m ON s.student_id = m.student_id
GROUP BY group_name;

Находит максимальную оценку по курсам каждого преподавателя
SELECT t.first_name, t.last_name, MAX(m.mark) as max_teacher_mark
FROM teachers t
JOIN courses c ON t.teacher_id = c.teacher_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY t.teacher_id, t.first_name, t.last_name;

Находит максимальную оценку по семестрам
SELECT semester_number, MAX(mark) as max_semester_mark
FROM semesters s
JOIN courses c ON s.semester_id = c.semester_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY semester_number;

Находит максимальную вместимость аудиторий по зданиям
SELECT building, MAX(capacity) as max_capacity
FROM classrooms
GROUP BY building;

Находит максимальную оценку по каждой дисциплине (дубликат для полноты)
SELECT subject_name, MAX(mark) as max_subject_mark
FROM subjects s
JOIN courses c ON s.subject_id = c.subject_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY subject_name;

Находит последний вход в систему по ролям
SELECT role_name, MAX(login_time) as last_login
FROM roles r
JOIN logins l ON r.role_id = l.role_id
GROUP BY role_name;

Находит максимальную оценку по факультетам
SELECT faculty_name, MAX(mark) as max_faculty_mark
FROM faculties f
JOIN departments d ON f.faculty_id = d.faculty_id
JOIN groups g ON d.department_id = g.department_id
JOIN students s ON g.group_id = s.group_id
JOIN marks m ON s.student_id = m.student_id
GROUP BY faculty_name;
```
Пример MIN:

Находит минимальную оценку для каждого студента
```sql
SELECT s.first_name, s.last_name, MIN(m.mark) as lowest_mark
FROM students s
JOIN marks m ON s.student_id = m.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```
![image](https://github.com/user-attachments/assets/adc70aa1-fc27-4ef0-9dfe-39c7bdf1dc6f)

Остальные MIN:

```sql
Находит минимальную оценку по каждой дисциплине
SELECT subject_name, MIN(mark) as lowest_subject_mark
FROM subjects s
JOIN courses c ON s.subject_id = c.subject_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY subject_name;

Находит минимальную вместимость аудиторий по кафедрам
SELECT department_name, MIN(capacity) as min_classroom_capacity
FROM departments d
JOIN classrooms c ON c.building IN (
    SELECT building
    FROM classrooms
    WHERE department_id = d.department_id
)
GROUP BY department_name;

Находит минимальную оценку по каждой группе
SELECT group_name, MIN(mark) as min_group_mark
FROM groups g
JOIN students s ON g.group_id = s.group_id
JOIN marks m ON s.student_id = m.student_id
GROUP BY group_name;

Находит минимальную оценку по курсам каждого преподавателя
SELECT t.first_name, t.last_name, MIN(m.mark) as min_teacher_mark
FROM teachers t
JOIN courses c ON t.teacher_id = c.teacher_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY t.teacher_id, t.first_name, t.last_name;

Находит минимальную оценку по семестрам
SELECT semester_number, MIN(mark) as min_semester_mark
FROM semesters s
JOIN courses c ON s.semester_id = c.semester_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY semester_number;

Находит минимальную вместимость аудиторий по зданиям
SELECT building, MIN(capacity) as min_capacity
FROM classrooms
GROUP BY building;

Находит минимальную оценку по каждой дисциплине (дубликат для полноты)
SELECT subject_name, MIN(mark) as min_subject_mark
FROM subjects s
JOIN courses c ON s.subject_id = c.subject_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY subject_name;

Находит первый вход в систему по ролям
SELECT role_name, MIN(login_time) as first_login
FROM roles r
JOIN logins l ON r.role_id = l.role_id
GROUP BY role_name;

Находит минимальную оценку по факультетам
SELECT faculty_name, MIN(mark) as min_faculty_mark
FROM faculties f
JOIN departments d ON f.faculty_id = d.faculty_id
JOIN groups g ON d.department_id = g.department_id
JOIN students s ON g.group_id = s.group_id
JOIN marks m ON s.student_id = m.student_id
GROUP BY faculty_name;
```
Пример SUM:

Вычисляет сумму оценок для каждого студента

```sql
SELECT s.first_name, s.last_name, SUM(m.mark) as total_marks
FROM students s
JOIN marks m ON s.student_id = m.student_id
GROUP BY s.student_id, s.first_name, s.last_name;
```
![image](https://github.com/user-attachments/assets/83327521-9ea9-4b2d-9600-f4a1b8ac104d)

Остальные SUM:

```sql
Вычисляет сумму оценок по каждой дисциплине
SELECT subject_name, SUM(mark) as total_subject_marks
FROM subjects s
JOIN courses c ON s.subject_id = c.subject_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY subject_name;

-- Вычисляет суммарную вместимость аудиторий по кафедрам
SELECT department_name, SUM(capacity) as total_classroom_capacity
FROM departments d
JOIN classrooms c ON c.building IN (
    SELECT building
    FROM classrooms
    WHERE department_id = d.department_id
)
GROUP BY department_name;

-- Вычисляет сумму оценок по каждой группе
SELECT group_name, SUM(mark) as total_group_marks
FROM groups g
JOIN students s ON g.group_id = s.group_id
JOIN marks m ON s.student_id = m.student_id
GROUP BY group_name;

-- Вычисляет сумму оценок по курсам каждого преподавателя
SELECT t.first_name, t.last_name, SUM(m.mark) as total_teacher_marks
FROM teachers t
JOIN courses c ON t.teacher_id = c.teacher_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY t.teacher_id, t.first_name, t.last_name;

-- Вычисляет сумму оценок по семестрам
SELECT semester_number, SUM(mark) as total_semester_marks
FROM semesters s
JOIN courses c ON s.semester_id = c.semester_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY semester_number;

-- Вычисляет суммарную вместимость аудиторий по зданиям
SELECT building, SUM(capacity) as total_capacity
FROM classrooms
GROUP BY building;

-- Вычисляет сумму оценок по каждой дисциплине (дубликат для полноты)
SELECT subject_name, SUM(mark) as total_subject_marks
FROM subjects s
JOIN courses c ON s.subject_id = c.subject_id
JOIN marks m ON c.course_id = m.course_id
GROUP BY subject_name;

-- Подсчитывает общее количество входов в систему по ролям
SELECT role_name, COUNT(*) as total_logins
FROM roles r
JOIN logins l ON r.role_id = l.role_id
GROUP BY role_name;

-- Вычисляет сумму оценок по факультетам
SELECT faculty_name, SUM(mark) as total_faculty_marks
FROM faculties f
JOIN departments d ON f.faculty_id = d.faculty_id
JOIN groups g ON d.department_id = g.department_id
JOIN students s ON g.group_id = s.group_id
JOIN marks m ON s.student_id = m.student_id
GROUP BY faculty_name;
```
## 6. Хранимые процедуры
Добавляет нового студента и логирует действие
```sql
CREATE OR REPLACE PROCEDURE add_student(
    p_first_name VARCHAR,
    p_last_name VARCHAR,
    p_group_id INTEGER
)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO students (first_name, last_name, group_id)
    VALUES (p_first_name, p_last_name, p_group_id);
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (currval('students_student_id_seq'), 1, CURRENT_TIMESTAMP, 'Добавлен студент');
    COMMIT;
END;
$$;
```
![image](https://github.com/user-attachments/assets/b9c88627-7054-4b8f-af52-a6d2c91bca49)
![image](https://github.com/user-attachments/assets/c0afc6dc-7c9d-451e-923f-22ef317caae2)

Добавляет оценку студенту и логирует действие

```SQL
CREATE OR REPLACE PROCEDURE update_student_mark(
    p_student_id INTEGER,
    p_course_id INTEGER,
    p_mark INTEGER
)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO marks (student_id, course_id, mark, date)
    VALUES (p_student_id, p_course_id, p_mark, CURRENT_DATE);
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (p_student_id, 1, CURRENT_TIMESTAMP, 'Обновлена оценка');
    COMMIT;
END;
$$;
```
![image](https://github.com/user-attachments/assets/6412b0e5-9860-4453-8877-c31629ba5bbd)

Регистрирует посещаемость студента в таблице attendances и логирует действие в logins

```sql
REATE OR REPLACE PROCEDURE record_attendance(
    p_student_id INTEGER,
    p_course_id INTEGER,
    p_status VARCHAR
)
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO attendances (student_id, course_id, date, status)
    VALUES (p_student_id, p_course_id, CURRENT_DATE, p_status);
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (p_student_id, 1, CURRENT_TIMESTAMP, 'Зарегистрирована посещаемость');
    RAISE NOTICE 'Посещаемость (%: %) зарегистрирована для студента с ID % на курсе с ID %', p_status, CURRENT_DATE, p_student_id, p_course_id;
    COMMIT;
END;
$$;
```
![image](https://github.com/user-attachments/assets/eb2941b3-daa0-4884-b7d7-4092b698a62d)

Планирует экзамен для курса в таблице exams и логирует действие в logins

```sql
CREATE OR REPLACE PROCEDURE schedule_exam(
    p_course_id INTEGER,
    p_classroom_id INTEGER,
    p_exam_date DATE,
    p_start_time TIME
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_teacher_id INTEGER;
BEGIN
    SELECT teacher_id INTO v_teacher_id FROM courses WHERE course_id = p_course_id;
    INSERT INTO exams (course_id, classroom_id, exam_date, start_time)
    VALUES (p_course_id, p_classroom_id, p_exam_date, p_start_time);
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (v_teacher_id, 2, CURRENT_TIMESTAMP, 'Запланирован экзамен');
    RAISE NOTICE 'Экзамен для курса с ID % запланирован в аудитории % на % в %', p_course_id, p_classroom_id, p_exam_date, p_start_time;
    COMMIT;
END;
$$;
```
![image](https://github.com/user-attachments/assets/a672753f-1218-4886-9228-996227e5a44b)

Переводит студента в другую группу в таблице students и логирует действие в logins

```sql
CREATE OR REPLACE PROCEDURE transfer_student(
    p_student_id INTEGER,
    p_new_group_id INTEGER
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE students
    SET group_id = p_new_group_id
    WHERE student_id = p_student_id;
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (p_student_id, 1, CURRENT_TIMESTAMP, 'Студент переведен');
    RAISE NOTICE 'Студент с ID % переведен в группу с ID %', p_student_id, p_new_group_id;
    COMMIT;
END;
$$;
```
![image](https://github.com/user-attachments/assets/a7a2a6e4-7b50-4454-b544-85f27225cd4b)

## 7. Триггеры

Логирует добавление нового студента в таблицу logins

```sql
CREATE OR REPLACE FUNCTION student_insert_trigger()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (NEW.student_id, 1, CURRENT_TIMESTAMP, 'Студент добавлен');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER student_after_insert
AFTER INSERT ON students
FOR EACH ROW
EXECUTE FUNCTION student_insert_trigger();
```
![image](https://github.com/user-attachments/assets/0fb1157a-ca1f-4631-908a-62e4015e6ca7)

Логирует обновление данных студента в таблицу logins

```sql
CREATE OR REPLACE FUNCTION student_update_trigger()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (NEW.student_id, 1, CURRENT_TIMESTAMP, 'Данные студента обновлены');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER student_after_update
AFTER UPDATE ON students
FOR EACH ROW
EXECUTE FUNCTION student_update_trigger();
```
![image](https://github.com/user-attachments/assets/0de9a3a8-06b7-40f4-adb2-e093e3e524b6)

Логирует удаление студента в таблицу logins
```sql
CREATE OR REPLACE FUNCTION student_delete_trigger()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (OLD.student_id, 1, CURRENT_TIMESTAMP, 'Студент удален');
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER student_after_delete
AFTER DELETE ON students
FOR EACH ROW
EXECUTE FUNCTION student_delete_trigger();
```
Логирует добавление новой оценки в таблицу logins

```sql
CREATE OR REPLACE FUNCTION mark_insert_trigger()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (NEW.student_id, 1, CURRENT_TIMESTAMP, 'Оценка добавлена');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER mark_after_insert
AFTER INSERT ON marks
FOR EACH ROW
EXECUTE FUNCTION mark_insert_trigger();
```
Логирует обновление оценки в таблицу logins
```sql
CREATE OR REPLACE FUNCTION mark_update_trigger()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (NEW.student_id, 1, CURRENT_TIMESTAMP, 'Оценка обновлена');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER mark_after_update
AFTER UPDATE ON marks
FOR EACH ROW
EXECUTE FUNCTION mark_update_trigger();
```
Логирует удаление оценки в таблицу logins
```sql
CREATE OR REPLACE FUNCTION mark_delete_trigger()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO logins (user_id, role_id, login_time, action)
    VALUES (OLD.student_id, 1, CURRENT_TIMESTAMP, 'Оценка удалена');
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER mark_after_delete
AFTER DELETE ON marks
FOR EACH ROW
EXECUTE FUNCTION mark_delete_trigger();
```
## Заключение

Все функции, запросы и триггеры протестированы и работают корректно.


