# Отчет по структуре базы данных университетской системы

Этот документ описывает структуру базы данных для управления университетом, реализованную в PostgreSQL. База включает 15 таблиц, охватывающих факультеты, кафедры, студентов, преподавателей, дисциплины и другие аспекты университетской деятельности. Ниже приведено описание каждой таблицы, ее внешних ключей и связей.

## Таблицы и их описание

### 1. `faculties` (Факультеты)
- **Описание**: Хранит информацию о факультетах.
- **Атрибуты**:
  - `faculty_id` (SERIAL, PRIMARY KEY): Уникальный ID факультета.
  - `faculty_name` (VARCHAR(100), NOT NULL): Название факультета.
- **Внешние ключи**: Нет.
- **Связи**: Один ко многим с `departments` (один факультет — много кафедр).

### 2. `departments` (Кафедры)
- **Описание**: Содержит данные о кафедрах.
- **Атрибуты**:
  - `department_id` (SERIAL, PRIMARY KEY): Уникальный ID кафедры.
  - `department_name` (VARCHAR(100), NOT NULL): Название кафедры.
  - `faculty_id` (INTEGER): Ссылка на `faculties(faculty_id)`.
- **Внешние ключи**: 
  - `faculty_id` → `faculties(faculty_id)`.
- **Связи**:
  - Многое к одному с `faculties`.
  - Один ко многим с `groups`, `teachers`, `subjects`.

### 3. `groups` (Учебные группы)
- **Описание**: Хранит данные об учебных группах.
- **Атрибуты**:
  - `group_id` (SERIAL, PRIMARY KEY): Уникальный ID группы.
  - `group_name` (VARCHAR(50), NOT NULL): Название группы.
  - `department_id` (INTEGER): Ссылка на `departments(department_id)`.
- **Внешние ключи**: 
  - `department_id` → `departments(department_id)`.
- **Связи**:
  - Многое к одному с `departments`.
  - Один ко многим с `students`.

### 4. `students` (Студенты)
- **Описание**: Содержит данные о студентах.
- **Атрибуты**:
  - `student_id` (SERIAL, PRIMARY KEY): Уникальный ID студента.
  - `first_name` (VARCHAR(50), NOT NULL): Имя.
  - `last_name` (VARCHAR(50), NOT NULL): Фамилия.
  - `group_id` (INTEGER): Ссылка на `groups(group_id)`.
- **Внешние ключи**: 
  - `group_id` → `groups(group_id)`.
- **Связи**:
  - Многое к одному с `groups`.
  - Один ко многим с `marks`, `attendances`, `logins`.

### 5. `teachers` (Преподаватели)
- **Описание**: Хранит данные о преподавателях.
- **Атрибуты**:
  - `teacher_id` (SERIAL, PRIMARY KEY): Уникальный ID преподавателя.
  - `first_name` (VARCHAR(50), NOT NULL): Имя.
  - `last_name` (VARCHAR(50), NOT NULL): Фамилия.
  - `department_id` (INTEGER): Ссылка на `departments(department_id)`.
- **Внешние ключи**: 
  - `department_id` → `departments(department_id)`.
- **Связи**:
  - Многое к одному с `departments`.
  - Один ко многим с `courses`, `logins`.

### 6. `subjects` (Учебные дисциплины)
- **Описание**: Содержит данные о дисциплинах.
- **Атрибуты**:
  - `subject_id` (SERIAL, PRIMARY KEY): Уникальный ID дисциплины.
  - `subject_name` (VARCHAR(100), NOT NULL): Название дисциплины.
  - `department_id` (INTEGER): Ссылка на `departments(department_id)`.
- **Внешние ключи**: 
  - `department_id` → `departments(department_id)`.
- **Связи**:
  - Многое к одному с `departments`.
  - Один ко многим с `courses`.

### 7. `semesters` (Семестры)
- **Описание**: Хранит данные о семестрах и учебных годах.
- **Атрибуты**:
  - `semester_id` (SERIAL, PRIMARY KEY): Уникальный ID семестра.
  - `semester_number` (INTEGER, NOT NULL): Номер семестра.
  - `academic_year` (VARCHAR(9), NOT NULL): Учебный год.
- **Внешние ключи**: Нет.
- **Связи**: Один ко многим с `courses`.

### 8. `courses` (Курсы)
- **Описание**: Связывает дисциплины, преподавателей и семестры.
- **Атрибуты**:
  - `course_id` (SERIAL, PRIMARY KEY): Уникальный ID курса.
  - `subject_id` (INTEGER): Ссылка на `subjects(subject_id)`.
  - `teacher_id` (INTEGER): Ссылка на `teachers(teacher_id)`.
  - `semester_id` (INTEGER): Ссылка на `semesters(semester_id)`.
- **Внешние ключи**:
  - `subject_id` → `subjects(subject_id)`.
  - `teacher_id` → `teachers(teacher_id)`.
  - `semester_id` → `semesters(semester_id)`.
- **Связи**:
  - Многое к одному с `subjects`, `teachers`, `semesters`.
  - Один ко многим с `marks`, `attendances`, `schedules`, `exams`.

### 9. `marks` (Оценки)
- **Описание**: Хранит оценки студентов по курсам.
- **Атрибуты**:
  - `mark_id` (SERIAL, PRIMARY KEY): Уникальный ID оценки.
  - `student_id` (INTEGER): Ссылка на `students(student_id)`.
  - `course_id` (INTEGER): Ссылка на `courses(course_id)`.
  - `mark` (INTEGER, CHECK 0-100): Оценка.
  - `date` (DATE, NOT NULL): Дата выставления.
- **Внешние ключи**:
  - `student_id` → `students(student_id)`.
  - `course_id` → `courses(course_id)`.
- **Связи**: Многое к одному с `students`, `courses`.

### 10. `attendances` (Посещаемость)
- **Описание**: Хранит данные о посещаемости студентов.
- **Атрибуты**:
  - `attendance_id` (SERIAL, PRIMARY KEY): Уникальный ID записи.
  - `student_id` (INTEGER): Ссылка на `students(student_id)`.
  - `course_id` (INTEGER): Ссылка на `courses(course_id)`.
  - `date` (DATE, NOT NULL): Дата занятия.
  - `status` (VARCHAR(20), CHECK): Статус ("присутствовал", "отсутствовал", "опоздал").
- **Внешние клю-Classroomчи**:
  - `student_id` → `students(student_id)`.
  - `course_id` → `courses(course_id)`.
- **Связи**: Многое к одному с `students`, `courses`.

### 11. `classrooms` (Аудитории)
- **Описание**: Содержит данные об аудиториях.
- **Атрибуты**:
  - `classroom_id` (SERIAL, PRIMARY KEY): Уникальный ID аудитории.
  - `room_number` (VARCHAR(20), NOT NULL): Номер аудитории.
  - `building` (VARCHAR(50), NOT NULL): Здание.
  - `capacity` (INTEGER, NOT NULL): Вместимость.
- **Внешние ключи**: Нет.
- **Связи**: Один ко многим с `schedules`, `exams`.

### 12. `schedules` (Расписание)
- **Описание**: Хранит расписание занятий.
- **Атрибуты**:
  - `schedule_id` (SERIAL, PRIMARY KEY): Уникальный ID записи.
  - `course_id` (INTEGER): Ссылка на `courses(course_id)`.
  - `classroom_id` (INTEGER): Ссылка на `classrooms(classroom_id)`.
  - `day_of_week` (VARCHAR(10), NOT NULL): День недели.
  - `start_time` (TIME, NOT NULL): Начало.
  - `end_time` (TIME, NOT NULL): Окончание.
- **Внешние ключи**:
  - `course_id` → `courses(course_id)`.
  - `classroom_id` → `classrooms(classroom_id)`.
- **Связи**: Многое к одному с `courses`, `classrooms`.

### 13. `exams` (Экзамены)
- **Описание**: Хранит данные об экзаменах.
- **Атрибуты**:
  - `exam_id` (SERIAL, PRIMARY KEY): Уникальный ID экзамена.
  - `course_id` (INTEGER): Ссылка на `courses(course_id)`.
  - `classroom_id` (INTEGER): Ссылка на `classrooms(classroom_id)`.
  - `exam_date` (DATE, NOT NULL): Дата экзамена.
  - `start_time` (TIME, NOT NULL): Время начала.
- **Внешние ключи**:
  - `course_id` → `courses(course_id)`.
  - `classroom_id` → `classrooms(classroom_id)`.
- **Связи**: Многое к одному с `courses`, `classrooms`.

### 14. `roles` (Роли пользователей)
- **Описание**: Хранит роли пользователей.
- **Атрибуты**:
  - `role_id` (SERIAL, PRIMARY KEY): Уникальный ID роли.
  - `role_name` (VARCHAR(50), NOT NULL): Название роли.
- **Внешние ключи**: Нет.
- **Связи**: Один ко многим с `logins`.

### 15. `logins` (Лог входов/действий)
- **Описание**: Хранит записи о действиях пользователей.
- **Атрибуты**:
  - `login_id` (SERIAL, PRIMARY KEY): Уникальный ID записи.
  - `user_id` (INTEGER, NOT NULL): ID пользователя (студент или преподаватель).
  - `role_id` (INTEGER): Ссылка на `roles(role_id)`.
  - `login_time` (TIMESTAMP, NOT NULL): Время действия.
  - `action` (VARCHAR(100), NOT NULL): Описание действия.
- **Внешние ключи**: 
  - `role_id` → `roles(role_id)`.
- **Связи**: Многое к одному с `roles`.

## Сводка связей
- `faculties` → `departments`: Один ко многим.
- `departments` → `groups`, `teachers`, `subjects`: Один ко многим.
- `groups` → `students`: Один ко многим.
- `students` → `marks`, `attendances`, `logins`: Один ко многим.
- `teachers` → `courses`, `logins`: Один ко многим.
- `subjects` → `courses`: Один ко многим.
- `semesters` → `courses`: Один ко многим.
- `courses` → `marks`, `attendances`, `schedules`, `exams`: Один ко многим.
- `classrooms` → `schedules`, `exams`: Один ко многим.
- `roles` → `logins`: Один ко многим.

## Примечания
- Внешние ключи обеспечивают целостность данных.
- Поле `user_id` в `logins` не имеет строгой ссылки на `students` или `teachers` для гибкости.
- Ограничения в `marks` (оценки 0-100) и `attendances` (определенные статусы) гарантируют корректность данных.