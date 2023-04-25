## Задание 1

### Создание
```SQL
USE my_users;
GO

CREATE TABLE Users_01 (
    last_name NVARCHAR(50),
    first_name NVARCHAR(50),
    middle_name NVARCHAR(50),
    birthday DATE,
    id int,
)
GO
```

### Заполнение
```Python
import csv
import random
import time
from datetime import datetime, timedelta

from russian_names import RussianNames


def get_users():
    rn = RussianNames(
        count=1000000, patronymic=True, transliterate=False, encoding="UTF-8"
    )
    data_to_insert = []

    for person in rn:
        first_name, middle_name, last_name = person.split()
        last_name = last_name
        first_name = first_name
        middle_name = middle_name
        random_date = start_date + timedelta(days=random.randint(0, delta.days))
        random_date = random_date.date()
        data_to_insert.append((last_name, first_name, middle_name, random_date))

    return data_to_insert


if __name__ == '__main__':
    st = time.time()
    start_date = datetime(1995, 1, 1)
    end_date = datetime(2005, 12, 31)
    delta = end_date - start_date

    data = []
    data.extend(get_users())

    with open('users2.csv', 'w', newline='', encoding='utf-8') as csvfile:
        fieldnames = ['last_name', 'first_name', 'middle_name', 'random_date']
        writer = csv.writer(csvfile)
        writer.writerow(fieldnames)
        writer.writerows(data)

    elapsed_time = time.time() - st
    print("Execution time:", time.strftime("%H:%M:%S", time.gmtime(elapsed_time)))
```

![Table data](/img/1.3_data.png)

### Копия таблицы
```SQL
USE my_users;
GO

CREATE TABLE Users_02 (
    last_name NVARCHAR(50),
    first_name NVARCHAR(50),
    middle_name NVARCHAR(50)
,    birthday DATE,
    id int NOT NULL PRIMARY KEY,
)

SET STATISTICS TIME ON

INSERT INTO Users_02
SELECT last_name, first_name, middle_name, birthday, id FROM Users_01

SET STATISTICS TIME OFF
GO
```
![Table copy](/img/1.2_copy_table.png)

## Задание 2

### а) Update
```SQL
USE my_users;
GO

ALTER TABLE Users_02 ADD full_age INT;
GO

UPDATE Users_02
SET Users_02.full_age = DATEDIFF(day, Users_02.birthday, GETDATE() - (DATEDIFF(year, Users_02.birthday, GETDATE()) / 4)) / 365;
GO
```
![Update full age](/img/2.1_update_time.png)

### б) Trigger
```SQL
USE my_users;

ALTER TABLE Users_03 ADD full_age INT;
GO

CREATE TRIGGER set_full_age
    ON Users_03
    AFTER INSERT AS
BEGIN
    UPDATE Users_03
    SET full_age = DATEDIFF(day, Users_03.birthday, GETDATE() - (DATEDIFF(year, Users_03.birthday, GETDATE()) / 4)) / 365
    WHERE full_age IS NULL;
END
GO

SET STATISTICS TIME ON

INSERT INTO Users_03 (last_name, first_name, middle_name, birthday, id)
VALUES (N'Фамилия', N'Имя', N'Отчество', N'2000-01-01', 1000001);

SET STATISTICS TIME OFF
GO
```
![Trigger full age](/img/2.2_trigger_time.png)

### в) Procedure
```SQL
USE my_users
GO

ALTER TABLE Users_04 ADD full_age INT;
GO

CREATE PROCEDURE SET_AGE AS
    UPDATE Users_04
    SET full_age = DATEDIFF(day, Users_04.birthday, GETDATE() - (DATEDIFF(year, Users_04.birthday, GETDATE()) / 4)) / 365;
GO

SET STATISTICS TIME ON

EXEC SET_AGE
GO

SET STATISTICS TIME OFF
GO
```
![Procedure full age](/img/2.3_procedure_time.png)

### Сводные данные
| Update    | Trigger | Procedure |
|---|---|---|
| 3.442 s   | 4.298 s | 3.364 s   |


## Задание 3
В созданную базу добавить поле "количество однофамильцев" и заполнить правильными значениями:
- a) через Update без создания индексов
- б) через Update с предварительным созданием индекса по полю с фамилией
- в) используя хранимую функцию подсчета количества однофамильцев
В каждом из случаев применяем код к исходной базе(отдельной копии) и замеряем время исполнения.
В пункте б) время считаем отдельно на создание индекса и на выполнение Update, потом суммируем


### а) Update без создания индексов

```SQL
USE my_users
GO

ALTER TABLE Users_02 ADD namesake_count INT;
GO

SET STATISTICS TIME ON

UPDATE Users_02
SET namesake_count = namesake_count_temp.namesake_count
FROM (SELECT last_name, count(last_name) namesake_count from Users_02 GROUP BY last_name) namesake_count_temp
WHERE Users_02.last_name = namesake_count_temp.last_name

SET STATISTICS TIME OFF
```
![Update without index](/img/3.1_update_without_index_time.png)

### б) через Update с предварительным созданием индекса по полю с фамилией

```SQL
USE my_users
GO

ALTER TABLE Users_03 ADD namesake_count INT;
GO

SET STATISTICS TIME ON
CREATE INDEX last_name_index ON Users_03(last_name);
SET STATISTICS TIME OFF

SET STATISTICS TIME ON

UPDATE Users_03
SET namesake_count = namesake_count_temp.namesake_count
FROM (SELECT last_name, count(last_name) namesake_count from Users_03 GROUP BY last_name) namesake_count_temp
WHERE Users_03.last_name = namesake_count_temp.last_name

SET STATISTICS TIME OFF
```
![Create index](/img/3.2_create_index.png)
![Update with index](/img/3.2_update_with_index_time.png)

### в) используя хранимую функцию подсчета количества однофамильцев

```SQL
USE my_users
GO

ALTER TABLE Users_04 ADD namesake_count INT;
GO

CREATE FUNCTION get_namesake_count(@lastname NVARCHAR(50)) RETURNS INT AS
    BEGIN
        DECLARE @result INT;
        SELECT @result = count(*) from Users_04 WHERE last_name = @lastname;
        RETURN @result;
    END;

SET STATISTICS TIME ON

UPDATE Users_04
SET Users_04.namesake_count = dbo.get_namesake_count(last_name);

SET STATISTICS TIME OFF
```
![Update with function](/img/3.3_function_time.png)

### Сводные данные

<table>
    <tbody>
        <tr>
            <td rowspan="3">Update</td>
            <td colspan="2">Update with index</td>
            <td rowspan="3">Function</td>
        </tr>
        <tr>
            <td>Index</td>
            <td>Update</td>
        </tr>
        <tr>
            <td>1.364 s</td>
            <td>4.689 s</td>
        </tr>
        <tr>
            <td>5.316 s</td>
            <td colspan="2">6.053 s</td>
            <td>17.937 s</td>
        </tr>
    </tbody>
</table>