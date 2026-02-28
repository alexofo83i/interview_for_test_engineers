## Создание таблиц для работы с вопросами
### создать таблицу с курсами
```sql
create table courses (
id INT primary key, 
name varchar (50)
);
```
### создать таблицу со студентами ( и ссылкой на курс )
```sql
create table students (
id INT primary key,
name varchar (100),
courses_id INT
);
```
## Заполнение таблиц начальными данными
### заполнить таблицу курсов
```sql
insert into courses (id, name)
values (1, 'Курс 1'), 
       (2, 'Курс 2'), 
       (3, 'Курс 3');
```
### заполнить таблицу студентов
```sql
INSERT INTO students (id, name, courses_id)
VALUES
  (1, 'Саша', 1),
  (2, 'Паша', 1),
  (3, 'Настя', 1),
  (4, 'Рита', NULL),
  (5, 'Даша', 2),
  (6, 'Артем', 2),
  (7, 'Сережа', 2),
  (8, 'Надя', 3),
  (9, 'Дима', 3),
  (10, 'Катя', 3);
```

## Вопросы по фильтрации данных и JOIN (50 шт)

### Фильтрация по ID и простые условия
1.  Найти студента с идентификатором, равным 5.
```sql
-- Вариант 1: Простая фильтрация WHERE
SELECT * FROM students WHERE id = 5;
```
```sql
-- Вариант 2: Использование IN для одного значения
SELECT * FROM students WHERE id IN (5);
```
```sql
-- Вариант 3: Использование FETCH FIRST (если нужно ограничить вывод)
SELECT * FROM students WHERE id = 5 FETCH FIRST 1 ROW ONLY;
```

2.  Вывести список курсов, у которых ID больше 1.
```sql
-- Вариант 1: Простое условие
SELECT * FROM courses WHERE id > 1;
```
```sql
-- Вариант 2: Использование BETWEEN (хотя тут больше подходит >)
SELECT * FROM courses WHERE id BETWEEN 2 AND 999999;
```
```sql
-- Вариант 3: Сортировка для наглядности
SELECT * FROM courses WHERE id > 1 ORDER BY id;
```

3.  Найти студентов, чей ID находится в диапазоне от 3 до 7 включительно.
```sql
-- Вариант 1: BETWEEN (стандартный способ)
SELECT * FROM students WHERE id BETWEEN 3 AND 7;
```
```sql
-- Вариант 2: Операторы сравнения
SELECT * FROM students WHERE id >= 3 AND id <= 7;
```
```sql
-- Вариант 3: Комбинация IN и BETWEEN (если диапазон некогерентный, но здесь нет)
SELECT * FROM students WHERE id IN (3,4,5,6,7);
```

4.  Вывести студентов, у которых ID не равен 2, 5 и 9.
```sql
-- Вариант 1: Оператор NOT IN
SELECT * FROM students WHERE id NOT IN (2, 5, 9);
```
```sql
-- Вариант 2: Комбинация AND с !=
SELECT * FROM students WHERE id != 2 AND id != 5 AND id != 9;
```
```sql
-- Вариант 3: Использование ALL (менее читаемо)
SELECT * FROM students WHERE id <> ALL (ARRAY[2, 5, 9]);
```

5.  Найти курс по его названию "Курс 2".
```sql
-- Вариант 1: Точное совпадение строки
SELECT * FROM courses WHERE name = 'Курс 2';
```
```sql
-- Вариант 2: Поиск по шаблону (если бы было не точное совпадение)
SELECT * FROM courses WHERE name LIKE 'Курс 2';
```
```sql
-- Вариант 3: С использованием ILIKE (регистронезависимый)
SELECT * FROM courses WHERE name ILIKE 'курс 2';
```

### Фильтрация по шаблону (LIKE)
6.  Найти всех студентов, чьи имена начинаются на букву 'С'.
```sql
-- Вариант 1: LIKE с шаблоном 'С%'
SELECT * FROM students WHERE name LIKE 'С%';
```
```sql
-- Вариант 2: ILIKE для регистронезависимости
SELECT * FROM students WHERE name ILIKE 'с%';
```
```sql
-- Вариант 3: Использование left() функции (менее производительно)
SELECT * FROM students WHERE left(name, 1) = 'С';
```

7.  Вывести студентов, в имени которых встречается сочетание 'аш' (например, 'Паша', 'Даша').
```sql
-- Вариант 1: LIKE с '%аш%'
SELECT * FROM students WHERE name LIKE '%аш%';
```
```sql
-- Вариант 2: POSITION (позиция подстроки)
SELECT * FROM students WHERE POSITION('аш' IN name) > 0;
```
```sql
-- Вариант 3: STRPOS (аналог POSITION в PostgreSQL)
SELECT * FROM students WHERE STRPOS(name, 'аш') > 0;
```

8.  Найти студентов, имена которых заканчиваются на 'а'.
```sql
-- Вариант 1: LIKE с '%а'
SELECT * FROM students WHERE name LIKE '%а';
```
```sql
-- Вариант 2: Использование right() функции
SELECT * FROM students WHERE right(name, 1) = 'а';
```
```sql
-- Вариант 3: Извлечение последнего символа через substring
SELECT * FROM students WHERE SUBSTRING(name FROM LENGTH(name) FOR 1) = 'а';
```

9.  Вывести студентов, чьи имена состоят ровно из 4 букв (для демонстрации использования LIKE с паттернами или функций работы со строками).
```sql
-- Вариант 1: LIKE с четырьмя подчеркиваниями
SELECT * FROM students WHERE name LIKE '____';
```
```sql
-- Вариант 2: Функция LENGTH
SELECT * FROM students WHERE LENGTH(name) = 4;
```
```sql
-- Вариант 3: CHAR_LENGTH (более корректно для символов Unicode)
SELECT * FROM students WHERE CHAR_LENGTH(name) = 4;
```

10. Найти все курсы, в названии которых есть цифра '3'.
```sql
-- Вариант 1: LIKE с '%3%'
SELECT * FROM courses WHERE name LIKE '%3%';
```
```sql
-- Вариант 2: SIMILAR TO (более сложный regex-подобный синтаксис)
SELECT * FROM courses WHERE name SIMILAR TO '%3%';
```
```sql
-- Вариант 3: Регулярное выражение POSIX
SELECT * FROM courses WHERE name ~ '3';
```

### Операторы IN / NOT IN
11. Вывести студентов, которые прикреплены к курсам с id 1 или 3.
```sql
-- Вариант 1: IN
SELECT * FROM students WHERE courses_id IN (1, 3);
```
```sql
-- Вариант 2: OR
SELECT * FROM students WHERE courses_id = 1 OR courses_id = 3;
```
```sql
-- Вариант 3: EXISTS с массивом (менее читаемо)
SELECT * FROM students WHERE courses_id = ANY (ARRAY[1, 3]);
```

12. Найти студентов, которые НЕ прикреплены к курсам с id 1 или 2.
```sql
-- Вариант 1: NOT IN (осторожно с NULL!)
SELECT * FROM students WHERE courses_id NOT IN (1, 2);
```
```sql
-- Вариант 2: AND с !=
SELECT * FROM students WHERE courses_id != 1 AND courses_id != 2;
```
```sql
-- Вариант 3: NOT IN с проверкой на NULL (безопасный вариант)
SELECT * FROM students WHERE courses_id NOT IN (1, 2) OR courses_id IS NULL;
-- Примечание: этот запрос вернет студента Рита (courses_id IS NULL), что может быть нежелательно.
-- Правильнее: исключить NULL отдельно, если нужно.
SELECT * FROM students WHERE courses_id IS NOT NULL AND courses_id NOT IN (1, 2);
```

13. Вывести список курсов, id которых есть в списке (1, 4, 5). (Обратите внимание, что курса с id 4 не существует).
```sql
-- Вариант 1: IN
SELECT * FROM courses WHERE id IN (1, 4, 5);
-- Вернет только курс с id = 1.
```
```sql
-- Вариант 2: OR
SELECT * FROM courses WHERE id = 1 OR id = 4 OR id = 5;
```
```sql
-- Вариант 3: ANY с массивом
SELECT * FROM courses WHERE id = ANY (ARRAY[1, 4, 5]);
```

14. Найти студентов, чьи имена встречаются в списке ('Саша', 'Миша', 'Даша').
```sql
-- Вариант 1: IN
SELECT * FROM students WHERE name IN ('Саша', 'Миша', 'Даша');
```
```sql
-- Вариант 2: OR
SELECT * FROM students WHERE name = 'Саша' OR name = 'Миша' OR name = 'Даша';
```
```sql
-- Вариант 3: ANY (для демонстрации)
SELECT * FROM students WHERE name = ANY (ARRAY['Саша', 'Миша', 'Даша']);
```

15. Вывести студентов, чьи имена отсутствуют в списке ('Рита', 'Надя', 'Катя').
```sql
-- Вариант 1: NOT IN
SELECT * FROM students WHERE name NOT IN ('Рита', 'Надя', 'Катя');
```
```sql
-- Вариант 2: AND с !=
SELECT * FROM students WHERE name != 'Рита' AND name != 'Надя' AND name != 'Катя';
```
```sql
-- Вариант 3: NOT ILIKE ALL (если нужна регистронезависимость)
SELECT * FROM students WHERE name NOT ILIKE ALL (ARRAY['Рита', 'Надя', 'Катя']);
-- ВНИМАНИЕ: NOT ILIKE ALL работает не так, как ожидается для списка значений.
-- Лучше использовать первый вариант.
```

### Операторы IS NULL / IS NOT NULL
16. Найти студентов, у которых не указан курс (courses_id is NULL).
```sql
-- Вариант 1: IS NULL
SELECT * FROM students WHERE courses_id IS NULL;
```
```sql
-- Вариант 2: Использование COALESCE (не для поиска, но для проверки)
SELECT * FROM students WHERE COALESCE(courses_id, -1) = -1;
-- Не рекомендуется, но возможно.
```
```sql
-- Вариант 3: В подзапросе с NOT EXISTS (избыточно)
SELECT * FROM students s WHERE NOT EXISTS (SELECT 1 FROM courses c WHERE c.id = s.courses_id);
```

17. Вывести всех студентов, которые закреплены за каким-либо курсом.
```sql
-- Вариант 1: IS NOT NULL
SELECT * FROM students WHERE courses_id IS NOT NULL;
```
```sql
-- Вариант 2: EXISTS
SELECT * FROM students s WHERE EXISTS (SELECT 1 FROM courses c WHERE c.id = s.courses_id);
```
```sql
-- Вариант 3: INNER JOIN (вернет только студентов с курсами)
SELECT s.* FROM students s JOIN courses c ON s.courses_id = c.id;
```

18. Придумать запрос, где условие проверяет, что имя студента не пустое (не NULL). (В текущих данных имен NULL нет, но запрос должен быть универсальным).
```sql
-- Вариант 1: IS NOT NULL для строки
SELECT * FROM students WHERE name IS NOT NULL;
```
```sql
-- Вариант 2: Проверка на пустую строку (если нужно исключить и пустые строки)
SELECT * FROM students WHERE name IS NOT NULL AND name != '';
```
```sql
-- Вариант 3: Использование LENGTH для проверки на пустоту (включая NULL)
SELECT * FROM students WHERE LENGTH(COALESCE(name, '')) > 0;
```

19. Найти курсы, у которых нет ни одного студента (используя подзапрос и IS NULL).
```sql
-- Вариант 1: Подзапрос с NOT IN и IS NULL (осторожно с NULL в подзапросе)
SELECT * FROM courses WHERE id NOT IN (SELECT DISTINCT courses_id FROM students WHERE courses_id IS NOT NULL);
-- Здесь важно исключить NULL из подзапроса.
```
```sql
-- Вариант 2: Подзапрос с NOT EXISTS (безопаснее)
SELECT * FROM courses c WHERE NOT EXISTS (SELECT 1 FROM students s WHERE s.courses_id = c.id);
```
```sql
-- Вариант 3: LEFT JOIN с проверкой IS NULL
SELECT c.* FROM courses c LEFT JOIN students s ON c.id = s.courses_id WHERE s.courses_id IS NULL;
```

20. Вывести студентов и название их курса, но если курс не указан, вывести 'Не зачислен'.
```sql
-- Вариант 1: LEFT JOIN с COALESCE
SELECT s.id, s.name, COALESCE(c.name, 'Не зачислен') AS course_name
FROM students s
LEFT JOIN courses c ON s.courses_id = c.id;
```
```sql
-- Вариант 2: CASE с LEFT JOIN
SELECT s.id, s.name, 
       CASE WHEN c.name IS NULL THEN 'Не зачислен' ELSE c.name END AS course_name
FROM students s
LEFT JOIN courses c ON s.courses_id = c.id;
```
```sql
-- Вариант 3: Скалярный подзапрос с COALESCE
SELECT s.id, s.name, 
       COALESCE((SELECT name FROM courses c WHERE c.id = s.courses_id), 'Не зачислен') AS course_name
FROM students s;
```

### Операторы EXISTS / NOT EXISTS (Semi-join / Anti-join)
21. Найти курсы, к которым прикреплен хотя бы один студент (semi-join через EXISTS).
```sql
-- Вариант 1: EXISTS
SELECT * FROM courses c WHERE EXISTS (SELECT 1 FROM students s WHERE s.courses_id = c.id);
```
```sql
-- Вариант 2: IN (может быть медленнее)
SELECT * FROM courses WHERE id IN (SELECT DISTINCT courses_id FROM students WHERE courses_id IS NOT NULL);
```
```sql
-- Вариант 3: INNER JOIN с DISTINCT (имитация semi-join)
SELECT DISTINCT c.* FROM courses c JOIN students s ON c.id = s.courses_id;
```

22. Вывести курсы, к которым не прикреплен ни один студент (anti-join через NOT EXISTS).
```sql
-- Вариант 1: NOT EXISTS
SELECT * FROM courses c WHERE NOT EXISTS (SELECT 1 FROM students s WHERE s.courses_id = c.id);
```
```sql
-- Вариант 2: NOT IN (с осторожностью)
SELECT * FROM courses WHERE id NOT IN (SELECT DISTINCT courses_id FROM students WHERE courses_id IS NOT NULL);
```
```sql
-- Вариант 3: LEFT JOIN с проверкой
SELECT c.* FROM courses c LEFT JOIN students s ON c.id = s.courses_id WHERE s.courses_id IS NULL;
```

23. Найти студентов, для которых существует курс с названием, содержащим цифру '1' (просто пример связки).
```sql
-- Вариант 1: EXISTS (связанный подзапрос)
SELECT * FROM students s WHERE EXISTS (SELECT 1 FROM courses c WHERE c.id = s.courses_id AND c.name LIKE '%1%');
```
```sql
-- Вариант 2: JOIN (если нужны поля из обеих таблиц)
SELECT s.*, c.name FROM students s JOIN courses c ON s.courses_id = c.id AND c.name LIKE '%1%';
```
```sql
-- Вариант 3: IN с подзапросом
SELECT * FROM students WHERE courses_id IN (SELECT id FROM courses WHERE name LIKE '%1%');
```

24. Вывести студентов, которые НЕ прикреплены к курсу с названием 'Курс 2' (используя NOT EXISTS).
```sql
-- Вариант 1: NOT EXISTS (корректно обрабатывает NULL)
SELECT * FROM students s WHERE NOT EXISTS (SELECT 1 FROM courses c WHERE c.id = s.courses_id AND c.name = 'Курс 2');
```
```sql
-- Вариант 2: LEFT JOIN с проверкой
SELECT s.* FROM students s LEFT JOIN courses c ON s.courses_id = c.id AND c.name = 'Курс 2' WHERE c.id IS NULL;
```
```sql
-- Вариант 3: NOT IN (осторожно с NULL)
SELECT * FROM students WHERE courses_id NOT IN (SELECT id FROM courses WHERE name = 'Курс 2');
-- Этот вариант исключит студентов с NULL courses_id.
```

25. Найти курсы, для которых существуют студенты с именем, начинающимся на 'С'.
```sql
-- Вариант 1: EXISTS
SELECT * FROM courses c WHERE EXISTS (SELECT 1 FROM students s WHERE s.courses_id = c.id AND s.name LIKE 'С%');
```
```sql
-- Вариант 2: INNER JOIN с DISTINCT
SELECT DISTINCT c.* FROM courses c JOIN students s ON c.id = s.courses_id AND s.name LIKE 'С%';
```
```sql
-- Вариант 3: IN с подзапросом
SELECT * FROM courses WHERE id IN (SELECT courses_id FROM students WHERE name LIKE 'С%' AND courses_id IS NOT NULL);
```

### Различные типы JOIN
#### INNER JOIN
26. Вывести идентификатор студента, его имя и название курса, на который он ходит.
```sql
-- Вариант 1: INNER JOIN (классика)
SELECT s.id, s.name, c.name AS course_name
FROM students s
INNER JOIN courses c ON s.courses_id = c.id;
```
```sql
-- Вариант 2: JOIN (по умолчанию INNER)
SELECT s.id, s.name, c.name AS course_name
FROM students s
JOIN courses c ON s.courses_id = c.id;
```
```sql
-- Вариант 3: USING (если поля называются одинаково)
SELECT s.id, s.name, c.name AS course_name
FROM students s
JOIN courses c ON s.courses_id = c.id;
-- USING не подойдет, так как поля называются courses_id и id.
```

27. Вывести только те пары "студент-курс", где название курса 'Курс 1'.
```sql
-- Вариант 1: INNER JOIN с условием WHERE
SELECT s.id, s.name, c.name
FROM students s
JOIN courses c ON s.courses_id = c.id
WHERE c.name = 'Курс 1';
```
```sql
-- Вариант 2: INNER JOIN с условием в JOIN
SELECT s.id, s.name, c.name
FROM students s
JOIN courses c ON s.courses_id = c.id AND c.name = 'Курс 1';
```
```sql
-- Вариант 3: Подзапрос с фильтрацией курса
SELECT s.id, s.name, c.name
FROM students s
JOIN (SELECT * FROM courses WHERE name = 'Курс 1') c ON s.courses_id = c.id;
```

#### LEFT / RIGHT / FULL OUTER JOIN
28. Вывести всех студентов и названия их курсов, включая студентов без курса.
```sql
-- Вариант 1: LEFT JOIN (студенты слева)
SELECT s.id, s.name, c.name AS course_name
FROM students s
LEFT JOIN courses c ON s.courses_id = c.id;
```
```sql
-- Вариант 2: RIGHT JOIN (поменяв таблицы местами)
SELECT s.id, s.name, c.name AS course_name
FROM courses c
RIGHT JOIN students s ON s.courses_id = c.id;
```
```sql
-- Вариант 3: LEFT JOIN с USING (не подходит из-за названий полей)
SELECT s.id, s.name, c.name AS course_name
FROM students s
LEFT JOIN courses c ON s.courses_id = c.id;
-- USING: SELECT ... FROM students LEFT JOIN courses ON courses_id = id;
```

29. Вывести все курсы и имена студентов, которые на них учатся, включая курсы без студентов. (Использовать RIGHT JOIN или LEFT JOIN, поменяв таблицы местами).
```sql
-- Вариант 1: LEFT JOIN (курсы слева)
SELECT c.name AS course_name, s.name AS student_name
FROM courses c
LEFT JOIN students s ON c.id = s.courses_id;
```
```sql
-- Вариант 2: RIGHT JOIN (студенты справа)
SELECT c.name AS course_name, s.name AS student_name
FROM students s
RIGHT JOIN courses c ON s.courses_id = c.id;
```
```sql
-- Вариант 3: LEFT JOIN с агрегацией (для демонстрации)
SELECT c.name AS course_name, string_agg(s.name, ', ') AS students
FROM courses c
LEFT JOIN students s ON c.id = s.courses_id
GROUP BY c.name;
```

30. Вывести все возможные пары "студент-курс", включая студентов без курса и курсы без студентов (FULL OUTER JOIN).
```sql
-- Вариант 1: FULL OUTER JOIN
SELECT s.id AS student_id, s.name AS student_name, c.id AS course_id, c.name AS course_name
FROM students s
FULL OUTER JOIN courses c ON s.courses_id = c.id;
```
```sql
-- Вариант 2: UNION LEFT и RIGHT (имитация FULL OUTER)
SELECT s.id AS student_id, s.name AS student_name, c.id AS course_id, c.name AS course_name
FROM students s
LEFT JOIN courses c ON s.courses_id = c.id
UNION
SELECT s.id AS student_id, s.name AS student_name, c.id AS course_id, c.name AS course_name
FROM students s
RIGHT JOIN courses c ON s.courses_id = c.id
WHERE s.id IS NULL; -- Только строки, которых не было в LEFT JOIN (курсы без студентов)
```
```sql
-- Вариант 3: FULL JOIN (сокращенный синтаксис)
SELECT s.id AS student_id, s.name AS student_name, c.id AS course_id, c.name AS course_name
FROM students s
FULL JOIN courses c ON s.courses_id = c.id;
```

#### CROSS JOIN
31. Получить все возможные комбинации имен студентов и названий курсов (декартово произведение).
```sql
-- Вариант 1: CROSS JOIN (явный)
SELECT s.name AS student_name, c.name AS course_name
FROM students s
CROSS JOIN courses c
ORDER BY s.name, c.name;
```
```sql
-- Вариант 2: FROM с запятой (неявный CROSS JOIN)
SELECT s.name AS student_name, c.name AS course_name
FROM students s, courses c
ORDER BY s.name, c.name;
```
```sql
-- Вариант 3: JOIN с условием 1=1
SELECT s.name AS student_name, c.name AS course_name
FROM students s
JOIN courses c ON 1=1
ORDER BY s.name, c.name;
```

32. Используя CROSS JOIN, создать список для проверки: каждый студент против каждого курса, но вывести только те комбинации, где студент не прикреплен к этому курсу.
```sql
-- Вариант 1: CROSS JOIN с условием WHERE
SELECT s.name AS student_name, c.name AS course_name
FROM students s
CROSS JOIN courses c
WHERE s.courses_id IS NULL OR s.courses_id != c.id
ORDER BY s.name, c.name;
-- Примечание: Этот вариант не совсем корректен, т.к. студент может быть прикреплен к курсу 1,
-- но этот запрос покажет его в комбинации с курсом 1, если у него courses_id = 1? 
-- Условие s.courses_id != c.id исключит курс 1 для студента с курсом 1.
-- Но IS NULL нужен для студентов без курса (они не прикреплены ни к одному курсу).
-- Да, все верно.
```
```sql
-- Вариант 2: CROSS JOIN с NOT EXISTS
SELECT s.name, c.name
FROM students s
CROSS JOIN courses c
WHERE NOT EXISTS (SELECT 1 FROM students s2 WHERE s2.id = s.id AND s2.courses_id = c.id);
-- Более явная проверка отсутствия прикрепления.
```
```sql
-- Вариант 3: CROSS JOIN с LEFT JOIN и проверкой
SELECT s.name, c.name
FROM students s
CROSS JOIN courses c
LEFT JOIN students s_check ON s_check.id = s.id AND s_check.courses_id = c.id
WHERE s_check.id IS NULL;
```

#### LATERAL JOIN
33. Для каждого курса найти первого студента по алфавиту, прикрепленного к этому курсу (используя LATERAL подзапрос).
```sql
-- Вариант 1: LATERAL с ORDER BY и LIMIT
SELECT c.name AS course_name, lat.name AS first_student
FROM courses c
LEFT JOIN LATERAL (
    SELECT s.name
    FROM students s
    WHERE s.courses_id = c.id
    ORDER BY s.name
    LIMIT 1
) lat ON true;
```
```sql
-- Вариант 2: LATERAL с DISTINCT ON (менее эффективно)
SELECT c.name AS course_name, lat.name AS first_student
FROM courses c
LEFT JOIN LATERAL (
    SELECT DISTINCT ON (s.courses_id) s.name
    FROM students s
    WHERE s.courses_id = c.id
    ORDER BY s.courses_id, s.name
) lat ON true;
```
```sql
-- Вариант 3: Коррелированный подзапрос (без LATERAL, но менее гибко)
SELECT c.name,
       (SELECT s.name FROM students s WHERE s.courses_id = c.id ORDER BY s.name LIMIT 1) AS first_student
FROM courses c;
```

34. Вывести список курсов и общее количество студентов на каждом курсе, но только для курсов, где студентов больше 2-х (используя LATERAL для предварительного подсчета).
```sql
-- Вариант 1: LATERAL с предварительным подсчетом
SELECT c.name, lat.student_count
FROM courses c
JOIN LATERAL (
    SELECT COUNT(*) AS student_count
    FROM students s
    WHERE s.courses_id = c.id
) lat ON lat.student_count > 2;
```
```sql
-- Вариант 2: LATERAL с HAVING (агрегация внутри LATERAL)
SELECT c.name, lat.student_count
FROM courses c
LEFT JOIN LATERAL (
    SELECT COUNT(*) AS student_count
    FROM students s
    WHERE s.courses_id = c.id
    GROUP BY s.courses_id
    HAVING COUNT(*) > 2
) lat ON true
WHERE lat.student_count IS NOT NULL;
```
```sql
-- Вариант 3: Традиционный GROUP BY с HAVING (без LATERAL)
SELECT c.name, COUNT(s.id) AS student_count
FROM courses c
LEFT JOIN students s ON c.id = s.courses_id
GROUP BY c.id, c.name
HAVING COUNT(s.id) > 2;
```

35. Для каждого студента вывести его имя и имя любого другого студента, который учится на том же курсе (используя LATERAL и само-join).
```sql
-- Вариант 1: LATERAL с исключением самого себя
SELECT s.name AS student, lat.name AS classmate
FROM students s
LEFT JOIN LATERAL (
    SELECT s2.name
    FROM students s2
    WHERE s2.courses_id = s.courses_id
      AND s2.id != s.id
    LIMIT 1
) lat ON true
WHERE s.courses_id IS NOT NULL;
```
```sql
-- Вариант 2: LATERAL с агрегацией всех одногруппников
SELECT s.name AS student, lat.classmates
FROM students s
LEFT JOIN LATERAL (
    SELECT string_agg(s2.name, ', ') AS classmates
    FROM students s2
    WHERE s2.courses_id = s.courses_id
      AND s2.id != s.id
) lat ON true
WHERE s.courses_id IS NOT NULL;
```
```sql
-- Вариант 3: Само-join (без LATERAL)
SELECT s1.name AS student, s2.name AS classmate
FROM students s1
JOIN students s2 ON s1.courses_id = s2.courses_id AND s1.id != s2.id
WHERE s1.courses_id IS NOT NULL
ORDER BY s1.name, s2.name;
```

### Комбинированные запросы
36. Найти студентов с именем, содержащим букву 'а', которые ходят на курс 'Курс 1'.
```sql
-- Вариант 1: JOIN с условиями
SELECT s.*
FROM students s
JOIN courses c ON s.courses_id = c.id
WHERE s.name LIKE '%а%' AND c.name = 'Курс 1';
```
```sql
-- Вариант 2: Подзапрос
SELECT *
FROM students
WHERE name LIKE '%а%' AND courses_id = (SELECT id FROM courses WHERE name = 'Курс 1');
```
```sql
-- Вариант 3: EXISTS
SELECT *
FROM students s
WHERE name LIKE '%а%' AND EXISTS (SELECT 1 FROM courses c WHERE c.id = s.courses_id AND c.name = 'Курс 1');
```

37. Вывести список курсов, у которых название не 'Курс 3', и к которым прикреплены студенты.
```sql
-- Вариант 1: EXISTS с условием
SELECT * FROM courses c
WHERE c.name != 'Курс 3'
AND EXISTS (SELECT 1 FROM students s WHERE s.courses_id = c.id);
```
```sql
-- Вариант 2: IN с подзапросом
SELECT * FROM courses
WHERE name != 'Курс 3'
AND id IN (SELECT DISTINCT courses_id FROM students WHERE courses_id IS NOT NULL);
```
```sql
-- Вариант 3: JOIN с DISTINCT
SELECT DISTINCT c.*
FROM courses c
JOIN students s ON c.id = s.courses_id
WHERE c.name != 'Курс 3';
```

38. Найти студентов, которые либо прикреплены к курсу 2, либо вообще не прикреплены ни к одному курсу.
```sql
-- Вариант 1: OR
SELECT * FROM students
WHERE courses_id = 2 OR courses_id IS NULL;
```
```sql
-- Вариант 2: UNION
SELECT * FROM students WHERE courses_id = 2
UNION
SELECT * FROM students WHERE courses_id IS NULL;
```
```sql
-- Вариант 3: CASE в WHERE (менее читаемо)
SELECT * FROM students
WHERE CASE WHEN courses_id IS NULL THEN true ELSE courses_id = 2 END;
```

39. Вывести названия курсов, у которых нет студентов с именами, начинающимися на 'С' (используя NOT EXISTS).
```sql
-- Вариант 1: NOT EXISTS
SELECT * FROM courses c
WHERE NOT EXISTS (SELECT 1 FROM students s WHERE s.courses_id = c.id AND s.name LIKE 'С%');
```
```sql
-- Вариант 2: LEFT JOIN с проверкой
SELECT c.*
FROM courses c
LEFT JOIN students s ON c.id = s.courses_id AND s.name LIKE 'С%'
WHERE s.id IS NULL;
```
```sql
-- Вариант 3: NOT IN
SELECT * FROM courses
WHERE id NOT IN (SELECT courses_id FROM students WHERE name LIKE 'С%' AND courses_id IS NOT NULL);
```

40. Найти студентов, которые ходят на те же курсы, что и 'Саша' (исключая самого 'Сашу').
```sql
-- Вариант 1: Подзапрос с IN
SELECT * FROM students
WHERE courses_id IN (SELECT courses_id FROM students WHERE name = 'Саша')
AND name != 'Саша';
```
```sql
-- Вариант 2: Само-join
SELECT s2.*
FROM students s1
JOIN students s2 ON s1.courses_id = s2.courses_id
WHERE s1.name = 'Саша' AND s2.name != 'Саша';
```
```sql
-- Вариант 3: EXISTS
SELECT * FROM students s2
WHERE EXISTS (SELECT 1 FROM students s1 WHERE s1.name = 'Саша' AND s1.courses_id = s2.courses_id)
AND s2.name != 'Саша';
```

41. Подсчитать количество студентов на каждом курсе, отобразить название курса и количество, включив курсы с нулем студентов.
```sql
-- Вариант 1: LEFT JOIN с GROUP BY
SELECT c.name, COUNT(s.id) AS student_count
FROM courses c
LEFT JOIN students s ON c.id = s.courses_id
GROUP BY c.id, c.name;
```
```sql
-- Вариант 2: Скалярный подзапрос
SELECT c.name,
       (SELECT COUNT(*) FROM students s WHERE s.courses_id = c.id) AS student_count
FROM courses c;
```
```sql
-- Вариант 3: LEFT JOIN с агрегатной функцией и COALESCE
SELECT c.name, COALESCE(COUNT(s.id), 0) AS student_count
FROM courses c
LEFT JOIN students s ON c.id = s.courses_id
GROUP BY c.id, c.name;
-- COUNT(s.id) автоматически считает только не-NULL, поэтому COALESCE необязателен.
```

42. Вывести имена студентов, которые учатся на курсе, где есть студент по имени 'Артем'.
```sql
-- Вариант 1: IN с подзапросом
SELECT name
FROM students
WHERE courses_id IN (SELECT courses_id FROM students WHERE name = 'Артем')
AND name != 'Артем'; -- Исключаем самого Артема, если нужно
```
```sql
-- Вариант 2: EXISTS
SELECT s2.name
FROM students s2
WHERE EXISTS (SELECT 1 FROM students s1 WHERE s1.name = 'Артем' AND s1.courses_id = s2.courses_id)
AND s2.name != 'Артем';
```
```sql
-- Вариант 3: JOIN
SELECT s2.name
FROM students s1
JOIN students s2 ON s1.courses_id = s2.courses_id
WHERE s1.name = 'Артем' AND s2.name != 'Артем';
```

43. Найти пары студентов, которые учатся на разных курсах (CROSS JOIN с условием).
```sql
-- Вариант 1: CROSS JOIN с условием
SELECT s1.name AS student1, s2.name AS student2
FROM students s1
CROSS JOIN students s2
WHERE s1.id < s2.id AND s1.courses_id != s2.courses_id;
```
```sql
-- Вариант 2: JOIN с неравным условием
SELECT s1.name AS student1, s2.name AS student2
FROM students s1
JOIN students s2 ON s1.id < s2.id AND s1.courses_id != s2.courses_id;
```
```sql
-- Вариант 3: CROSS JOIN с исключением равных курсов и дублей
SELECT DISTINCT
    LEAST(s1.name, s2.name) AS student1,
    GREATEST(s1.name, s2.name) AS student2
FROM students s1, students s2
WHERE s1.courses_id != s2.courses_id AND s1.name != s2.name;
```

44. Вывести для каждого студента его имя и количество студентов на его курсе (используя LATERAL или оконные функции).
```sql
-- Вариант 1: Оконная функция (более эффективно)
SELECT name, courses_id, 
       COUNT(*) OVER (PARTITION BY courses_id) AS students_on_course
FROM students
WHERE courses_id IS NOT NULL
UNION ALL
SELECT name, NULL, 0 FROM students WHERE courses_id IS NULL;
-- Отдельно обрабатываем NULL, так как PARTITION BY NULL сгруппирует всех.
```
```sql
-- Вариант 2: LATERAL
SELECT s.name, lat.cnt
FROM students s
LEFT JOIN LATERAL (
    SELECT COUNT(*) AS cnt
    FROM students s2
    WHERE s2.courses_id = s.courses_id
) lat ON true
ORDER BY s.name;
```
```sql
-- Вариант 3: Коррелированный подзапрос
SELECT name,
       (SELECT COUNT(*) FROM students s2 WHERE s2.courses_id = s.courses_id) AS students_on_course
FROM students s;
```

45. Найти курсы, у которых максимальный ID среди всех курсов (подзапрос).
```sql
-- Вариант 1: Подзапрос с MAX
SELECT * FROM courses
WHERE id = (SELECT MAX(id) FROM courses);
```
```sql
-- Вариант 2: ORDER BY с LIMIT
SELECT * FROM courses
ORDER BY id DESC
LIMIT 1;
```
```sql
-- Вариант 3: Подзапрос с ALL
SELECT * FROM courses
WHERE id >= ALL (SELECT id FROM courses);
```

46. Вывести студента с минимальным ID, который учится на курсе 'Курс 2'.
```sql
-- Вариант 1: Подзапрос с MIN
SELECT * FROM students
WHERE courses_id = (SELECT id FROM courses WHERE name = 'Курс 2')
ORDER BY id
LIMIT 1;
```
```sql
-- Вариант 2: Подзапрос с MIN по ID
SELECT * FROM students
WHERE id = (SELECT MIN(id) FROM students WHERE courses_id = (SELECT id FROM courses WHERE name = 'Курс 2'));
```
```sql
-- Вариант 3: JOIN с подзапросом
SELECT s.*
FROM students s
JOIN courses c ON s.courses_id = c.id AND c.name = 'Курс 2'
ORDER BY s.id
LIMIT 1;
```

47. Обновить (мысленно) название курса для студента 'Рита' на 'Курс 3', используя UPDATE с JOIN.
```sql
-- Вариант 1: UPDATE с FROM (синтаксис PostgreSQL)
UPDATE students
SET courses_id = c.id
FROM courses c
WHERE students.name = 'Рита' AND c.name = 'Курс 3';
-- Проверка:
-- SELECT * FROM students WHERE name = 'Рита';
```
```sql
-- Вариант 2: UPDATE с подзапросом (ANSI)
UPDATE students
SET courses_id = (SELECT id FROM courses WHERE name = 'Курс 3')
WHERE name = 'Рита';
```
```sql
-- Вариант 3: UPDATE с USING (альтернативный синтаксис)
UPDATE students
SET courses_id = courses.id
FROM courses
WHERE students.name = 'Рита' AND courses.name = 'Курс 3';
```

48. Удалить (мысленно) всех студентов, которые не прикреплены ни к одному курсу, используя подзапрос с ID.
```sql
-- Вариант 1: DELETE с подзапросом (на основе NULL)
DELETE FROM students
WHERE id IN (SELECT id FROM students WHERE courses_id IS NULL);
```
```sql
-- Вариант 2: DELETE с подзапросом (на основе NOT EXISTS)
DELETE FROM students s
WHERE NOT EXISTS (SELECT 1 FROM courses c WHERE c.id = s.courses_id);
```
```sql
-- Вариант 3: Простой DELETE (без подзапроса)
DELETE FROM students WHERE courses_id IS NULL;
```

49. Найти студентов, у которых имя совпадает с именем другого студента (само-join).
```sql
-- Вариант 1: Само-join
SELECT DISTINCT s1.name
FROM students s1
JOIN students s2 ON s1.name = s2.name AND s1.id != s2.id;
-- Находит имена, которые повторяются.
```
```sql
-- Вариант 2: GROUP BY с HAVING
SELECT name
FROM students
GROUP BY name
HAVING COUNT(*) > 1;
```
```sql
-- Вариант 3: EXISTS с корреляцией
SELECT DISTINCT name
FROM students s1
WHERE EXISTS (SELECT 1 FROM students s2 WHERE s2.name = s1.name AND s2.id != s1.id);
```

50. Вывести все курсы и для каждого курс самый длинный (по длине имени) среди студентов на этом курсе.
```sql
-- Вариант 1: LATERAL с ORDER BY и LIMIT
SELECT c.name AS course, lat.longest_name
FROM courses c
LEFT JOIN LATERAL (
    SELECT s.name AS longest_name
    FROM students s
    WHERE s.courses_id = c.id
    ORDER BY LENGTH(s.name) DESC, s.name
    LIMIT 1
) lat ON true;
```
```sql
-- Вариант 2: DISTINCT ON с LATERAL (альтернатива)
SELECT c.name AS course, lat.name AS longest_name
FROM courses c
LEFT JOIN LATERAL (
    SELECT DISTINCT ON (s.courses_id) s.name
    FROM students s
    WHERE s.courses_id = c.id
    ORDER BY s.courses_id, LENGTH(s.name) DESC, s.name
) lat ON true;
```
```sql
-- Вариант 3: Оконная функция с ранжированием
SELECT course, name
FROM (
    SELECT c.name AS course, s.name,
           ROW_NUMBER() OVER (PARTITION BY c.id ORDER BY LENGTH(s.name) DESC, s.name) AS rn
    FROM courses c
    LEFT JOIN students s ON c.id = s.courses_id
) ranked
WHERE rn = 1;
```
