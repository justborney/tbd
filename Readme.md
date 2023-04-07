## Задание 1

### Создание
```SQL
USE my_users;
GO

CREATE TABLE Users_1 (
    first_name VARCHAR(50) COLLATE Latin1_General_100_CI_AI_SC_UTF8,
    last_name VARCHAR(50) COLLATE Latin1_General_100_CI_AI_SC_UTF8,
    middle_name VARCHAR(50) COLLATE Latin1_General_100_CI_AI_SC_UTF8,
    birthday DATE,
    id int NOT NULL IDENTITY(1, 1) PRIMARY KEY,
)
GO
```

### Заполнение
```Python
import random
import time
from datetime import datetime, timedelta

import pyodbc
from russian_names import RussianNames


def get_users():
    rn = RussianNames(
        count=100000, patronymic=True, transliterate=False, encoding="UTF-8"
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

    cursor.executemany(
        """
        INSERT INTO dbo.Users_1 (last_name, first_name, middle_name, birthday)
        VALUES (?, ?, ?, ?)""",
        data_to_insert,
    )
    conn.commit()


if __name__ == '__main__':
    st = time.time()
    start_date = datetime(1995, 1, 1)
    end_date = datetime(2005, 12, 31)
    delta = end_date - start_date

    # Connection
    server = "10.10.29.125,1433"
    database = "my_users"
    username = "sa"
    password = "admin"
    conn = pyodbc.connect(
        f"""
        DRIVER=ODBC Driver 18 for SQL Server;
        SERVER={server};
        DATABASE={database};
        UID={username};
        PWD={password};
        Encrypt=no;
        """
    )
    cursor = conn.cursor()

    for i in range(10):
        get_users()

    elapsed_time = time.time() - st
    print("Execution time:", time.strftime("%H:%M:%S", time.gmtime(elapsed_time)))
```

### Копия таблицы
```SQL
USE my_users;
GO

CREATE TABLE Users_2 (
    first_name VARCHAR(50) COLLATE Latin1_General_100_CI_AI_SC_UTF8,
    last_name VARCHAR(50) COLLATE Latin1_General_100_CI_AI_SC_UTF8,
    middle_name VARCHAR(50) COLLATE Latin1_General_100_CI_AI_SC_UTF8,
    birthday DATE,
    id int NOT NULL PRIMARY KEY,
)
GO

INSERT INTO Users_2
SELECT * FROM Users_1
GO
```

## Задание 2

### Пункт а) Update
```SQL
USE my_users;
GO

ALTER TABLE Users_2 ADD full_age INT;
GO

UPDATE Users_2
SET Users_2.full_age = DATEDIFF(day, Users_2.birthday, GETDATE() - (DATEDIFF(year, Users_2.birthday, GETDATE()) / 4)) / 365;
GO
```

### Пункт б) Trigger
```SQL
USE my_users;
GO

SET STATISTICS TIME ON

CREATE TRIGGER set_full_age
    ON Users_2
    AFTER INSERT AS
BEGIN
    UPDATE Users_2
    SET full_age = DATEDIFF(day, Users_2.birthday, GETDATE() - (DATEDIFF(year, Users_2.birthday, GETDATE()) / 4)) / 365
    WHERE full_age IS NULL;
END
GO

SET STATISTICS TIME OFF
GO
```

### Пункт в) Procedure
```SQL
USE my_users
GO

SET STATISTICS TIME ON

ALTER TABLE Users_3 ADD full_age INT;
GO

CREATE PROCEDURE SET_AGE AS
    UPDATE Users_3
    SET full_age = DATEDIFF(day, Users_3.birthday, GETDATE() - (DATEDIFF(year, Users_3.birthday, GETDATE()) / 4)) / 365;
GO

EXEC SET_AGE
GO

SET STATISTICS TIME OFF
GO
```