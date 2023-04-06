-- 1 Задание

-- Создание 
CREATE TABLE User (
    id int NOT NULL IDENTITY(1, 1) PRIMARY KEY,
    first_name VARCHAR(50) COLLATE Latin1_General_100_CI_AI_SC_UTF8,
    last_name VARCHAR(50) COLLATE Latin1_General_100_CI_AI_SC_UTF8,
    middle_name VARCHAR(50) COLLATE Latin1_General_100_CI_AI_SC_UTF8,
    birthday DATE
)

-- Заполнение
```
import random
from datetime import datetime, timedelta

import pyodbc
from russian_names import RussianNames

start_date = datetime(1995, 1, 1)
end_date = datetime(2005, 12, 31)
delta = end_date - start_date

# Connection
server = '10.10.29.125,1433'
database = 'my_users'
username = 'sa'
password = 'admin'
conn = pyodbc.connect(
    'DRIVER={ODBC Driver 18 for SQL Server};SERVER=' + 
    server + ';DATABASE=' + database + ';UID=' + 
    username + ';PWD=' + password + ';Encrypt=no;'
)
cursor = conn.cursor()

rn = RussianNames(count=1, patronymic=True, transliterate=False, encoding='UTF-8')
for person in rn:
    first_name, last_name, second_name = person.split()
    first_name = first_name
    last_name = last_name
    second_name = second_name
    random_date = start_date + timedelta(days=random.randint(0, delta.days))
    random_date = random_date.date()
    cursor.execute(
        """
        INSERT INTO dbo.Users (first_name, middle_name, last_name, birthday)
        VALUES (?, ?, ?, ?)""",
        first_name, last_name, second_name, random_date
    )
    conn.commit()
```
