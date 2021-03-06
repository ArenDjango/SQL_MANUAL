# Быстрый справочник по SQL (Beta version)

Навигация
- Очередь блоков
- Агрегатные функции
- HAVING
- JOIN
- Транзакции
- Индексы
- Блокировки
- SQL Иньекция
- Предстовление VIEW
- Триггеры

## #Очередь блоков
- SELECT
- JOIN
- WHERE
- GROUP BY 
- HAVING
- ORDER BY

## #Агрегатные функции
- MIN - Возвращает наименьшее значение выбранного столбца.
- MIN - Возвращает наибольшее значение выбранного столбца.
- COUNT - Функция возвращает количество строк , которое соответствует заданному критерию.
- AVG - Функция возвращает среднее значение числового столбца.
- SUM - Функция возвращает общую сумму числового столбца. 

## #HAVING
Предложение HAVING было добавлено в SQL, поскольку ключевое слово WHERE нельзя использовать с агрегатными функциями.
### Пример:
HAVING COUNT(CustomerID) > 5;

## #JOIN
#### CROSS JOIN 
Это когда вместо JOIN пишут SELECT что то from table1, table2, Не стоит использовать
#### INNER JOIN
Выбирает записи, которые имеют соответствующие значения в обеих таблицах.
#### LEFT JOIN
Возвращает все записи из левой таблицы (табл.1), а также совпадающие записи из таблицы справа (table2). Результатом будет 0 записей с правой стороны, если совпадений нет.

LEFT JOIN возвращает все записи из левой таблицы (Клиенты), даже если в правой таблице (Заказы) нет совпадений.

[id, title, order_id] [null or order data]
#### RIGHT JOIN
 Возвращает все записи из таблицы справа (table2) и совпадающие записи из левой таблицы (table1). Результатом будет 0 записей с левой стороны, если совпадений нет.
 RIGHT JOIN возвращает все записи из правой таблицы (Сотрудники), даже если в левой таблице (Заказы) нет совпадений.
 
 [null or order data] [id, title, order_id]
 
## #Транзакции
 Транхакции нужны когда группы запросов должны выполняться в месте, и если один из них пройдет не успешно то все изменении для всех этих таблиц откатываються
 
 ### SQLite, PostgreSQL
    BEGIN
      Группа запросов
    COMMIT
 ### MYSQL
    START_TRANSACTION
      Группа запросов
    COMMIT
  
Если нужно не сохранять результат в место COMMIT пишем ROLLBACK

SAVEPOINT point_name создать точку

ROLLBACK TO SAVEPOINT point_name возврат к точку

## #Индексы
Индексы работают как справочник у книги, отсартированному по альфавиту, поиск по отсартированному справочнику найти гараздо легче чем листать каждую страницу

Индексы работают так же, он берет и создает клон этой колонки и сортирует, при поиске он ищет по этой отсартированной колонке

Не стоит ставить индексы если у вас мало строк у таблиц, если десятки тысяч и больше то стоит если вы часто делаете запросы на чтения используя эти блоки

    WHERE
    ORDER BY 
    ...
    
    
    CREATE INDEX age ON users(age);
    
    CREATE UNIQUE INDEX email ON users(email)

В mysql индексы работают по алгоритму b-tree (это не бинарное дерево)

#### Составный индексы

КОгда делаеться запрос на чтение сразу по нескольким полям и нескольким WHERE, тогда ставиться индекс по сразу нескольким полям, здесь главное очередь,
например есть запрос

    WHERE title = 'example' AND gender = 'M' 

лучше ставить первым у составного индексы title, потамучто он повторяеться реже чем gender

    CREATE INDEX title_gender ON users(title, gender);

Так следует проверить работу запроса и индекса

    EXPLAIN SELECT * FROM users WHERE email = 'golotyuk@gmail.com';


## #Блокировки
MySQL от имени одного из клиентов накладывает блокировку на определенный ресурс, при этом другие клиенты ждут освобождения блокировки. Блокировка может быть на уровне таблиц (блокируется таблица) или на уровне строк (блокируются определенные строки таблицы). В механизме хранения MyISAM (используемом по умолчанию) реализована табличная блокировка, а в механизме InnoDB построчная. Построчная блокировка достигается посредством усложнения структуры хранилища: в MyISAM структура файла с данными представляет собой простое перечисление строк таблицы, тогда как хранилище InnoDB структурировано и поддерживает мультиверсионность данных. Поэтому, InnoDB выигрывает в приложениях, в которых происходит многопоточное изменение данных в одну и ту же таблицу, несмотря на необходимые потери на обслуживание более сложного хранилища.

#### Блокировки бывают двух видов: на чтение и на запись.
Если A хочет читать данные, то другие клиенты тоже могут читать данные, но никто не может записывать, пока А не закончит чтение (read lock).
Если А хочет записать данные, то другие клиенты не должны ни читать ни писать эти данные пока А не закончит(write lock).
 
 
#### Блокировка может быть наложена явно или неявно.
Если клиент не назначает блокировку явным образом, MySQL сервер неявно устанавливает необходимый тип блокировки на время выполнения выражения или транзакции. Например, в случае выполнения оператора SELECT сервер установит READ LOCK, а в случае UPDATE — WRITE LOCK. При неявной блокировке уровень блокировки зависит от типа хранилища данных: для MyISAM, MEMORY и MERGE блокируется вся таблица, для InnoDB — только используемые в выражении строки (в случае, если набор этих строк может быть однозначно определен значениями первичного ключа — иначе, блокируется вся таблица).

Часто возникает необходимость выполнения нескольких запросов подряд без вмешательства других клиентов в это время. Неявная блоктровка не подходит для этих целей, так как устанавливается только на время выполнения одного запроса. В этом случае клиент может явно назначить, а потом отменить блокировку с помощью выражений LOCK TABLES и UNLOCK TABLES. Явной блокировка всегда блокирует всю таблицу, независимо от механизма хранения.
 
 
Так как у тебя механизм хранения InnoDB, то неявная блокировка будет происходить на уровне строк.
Если ты хочешь что бы твои данные были правильно записаны и считаны, то перед записью этих данных явно установи блокировку (Устанавливается на всю таблицу). А после сними ее. Пример:

    LOCK TABLES clients READ LOCAL;
    ... производим запись (insert, update)...
    UNLOCK TABLES;
    
Здесь:

LOCK TABLES - задает блокировку таблицы
clients - имя блокируемой таблицы
READ LOCAL - тип блокировки (блокирует таблицу для чтения, но позволяет осуществлять вставку данных)
 
## #SQL Иньекция
Когда хакер может предугадать какой sql код написал прогер и может менять этот запрос внедряя поле свои символы типа ', '', or, или другие специальные символы

не нужно ставить переменную сразу в запрос а нужно фильтровать с перва, у разных ЯП свои функции

#### На примере PHP
- Нужно ставить типы и использовать DTO, на пример числа преоброзовать тип int и тд
- Экранирование значении mysqli_real_escape_string($var)


 
