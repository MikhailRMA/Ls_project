# Анализ клиентов банка

#### 1 Создать базу данных, таблицы (clients, creditscore, creditcard, product) и загрузить соответствующую информацию
```sql
CREATE DATABASE mydb;
USE mydb;
CREATE TABLE client(CustomerId INT  PRIMARY KEY AUTO_INCREMENT,
                  Surname VARCHAR(30),
                  Geography VARCHAR(40),
                  Gender VARCHAR(15),
                  Age INT,
                  Tenure INT,
                  Balance DECIMAL(8,2), 
                  EstimatedSalary DECIMAL(9,2),
                  Exited INT,
                  Complain INT,
                  Satisfaction_Score INT,
                  Point_Earned INT);
```


```sql
LOAD DATA LOCAL INFILE 'C:/db/client.csv'
INTO TABLE client
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```
Таблица client и ее заполнение происходило по средствам команд Mysql
Остальные таблицы созданы по средствам инструментария в среде Mysql Worckbench
```text

# CustomerId	Surname	Geography	Gender	Age	Tenure	Balance	EstimatedSalary	Exited	Complain	Satisfaction_Score	Point_Earned
15565701	Ferri	Spain	Male	32	2	76087.98	151822.66	0	0	1	308
15565779	Kent	France	Male	28	7	0.00	823.96	0	0	1	383
15565796	Docherty	Spain	Female	37	3	99773.85	54865.92	0	0	2	403
15565714	Cattaneo	Spain	Male	39	9	0.00	68873.80	0	0	5	535
15565706	Akobundu	France	Female	28	4	141792.61	22001.91	0	0	2	695
```
#### 2 Коллеги предоставили файл csv с неполной информацией заказанной выгрузки, отсутствуют важные данные, как активно клиент пользуется услугами банка. Обновить таблицу client и добавить столбец IsActiveMember из новой выгрузки
```sql
ALTER TABLE client
ADD IsActiveMember INT;
```
```text
Добавили новый столбец и наполнили данными.
# CustomerId	Surname	Geography	Gender	Age	Tenure	Balance	EstimatedSalary	Exited	Complain	Satisfaction Score	Point Earned	IsActiveMember
15565701	Ferri	Spain	Male	32	2	76087.98	151822.66	0	0	1	308	1
15565706	Akobundu	France	Female	28	4	141792.61	22001.91	0	0	2	695	1
15565714	Cattaneo	Spain	Male	39	9	0	68873.8	0	0	5	535	0
15565779	Kent	France	Male	28	7	0	823.96	0	0	1	383	0
15565796	Docherty	Spain	Female	37	3	99773.85	54865.92	0	0	2	403	0
```
#### 3 Отобразите действующих пользователей банка

```sql
SELECT CustomerId, Surname 
FROM client
WHERE Exited = 0; # WHERE Exited LIKE 0;
```
```text
# CustomerId	Surname
15565701	Ferri
15565706	Akobundu
15565714	Cattaneo
15565779	Kent
15565796	Docherty
```
#### 4 Показать активных клиентов из Франции и отсортировать их по возрасту.

```sql
SELECT CustomerId, Surname, age 
FROM client
WHERE Geography = 'France' AND exited = 0               
ORDER BY Age
LIMIT 5;
```
```text
# CustomerId	Surname	age
15706458	Pan	18
15689781	Ts'ai	18
15722611	Cameron	18
15597467	Duncan	18
15632472	Scott	18
```
#### 4 Для проведения новой акции требуется найти людей из Испании в возрастной группе от 25 до 30 лет, которые не очень активно пользуются продуктом банка. Отсортировать по снижению дохода клиента.
```sql
SELECT CustomerId, Surname, age, IsActiveMember,  EstimatedSalary 
FROM client
WHERE Geography = Spain AND IsActiveMember = 0 AND age BETWEEN 25 AND 30
ORDER BY EstimatedSalary DESC
LIMIT 5;
```
```text
# CustomerId	Surname	age	IsActiveMember	EstimatedSalary
15692671	Dobson	30	0	199099.51
15655875	Thao	28	0	198251.52
15642394	He	28	0	197588.32
15611058	Eluemuno	30	0	196582.28
15770041	Manna	29	0	192120.66
```
#### 5 В какой стране подано больше всего жалоб на работу продукта
```sql
SELECT Geography, COUNT(Complain = 1) AS Количество_жалоб
FROM client
GROUP BY Geography
ORDER BY Количество_жалоб;
```
```text
# Geography	Количество_жалоб
Spain	2477
Germany	2508
France	5014
```
#### 6 Посчитать общее количество клиентов для каждой страны и найти процент от общего числа клиентов.
```sql
SELECT Geography, COUNT(CustomerId) AS Количество_клиентов, (COUNT(CustomerId) / (SELECT COUNT(*) FROM client)) * 100 AS Процент_от_общего_числа
FROM client
WHERE exited = 0
GROUP BY Geography
ORDER BY Процент_от_общего_числа DESC;
```
```text
# Geography	Количество_клиентов	Процент_от_общего_числа
France	4203	42.0342
Spain	2064	20.6421
Germany	1695	16.9517
```
#### 7 В какой стране произошел большой отток клиентов
```sql
SELECT Geography, COUNT(CustomerId) AS Количество_клиентов
FROM client
WHERE exited = 1
GROUP BY Geography
ORDER BY Количество_клиентов DESC;
```
```text
# Geography	Количество_клиентов
Germany	813
France	811
Spain	413
```
#### 8 В Германии большой отток клиентов и жалоб на сервис. Чтобы понять, что не так с сервисом выяснить с каким стажем использования продуктов уходят клиенты.
```sql
SELECT Tenure, COUNT(Tenure) AS Количество
FROM client
WHERE Geography = 'Germany'  AND exited = 1
GROUP BY Tenure
ORDER BY Tenure ASC;
```
```text
# Tenure	Количество
0	170
1	446
2	435
3	408
4	427
5	409
6	426
7	476
8	396
9	415
10	195
```
#### 9 Разделить клиентов на категории по стажу использования сервисов банка от 0-3, 4-7, 8-10.
```sql
SELECT *,
  CASE 
    WHEN Tenure BETWEEN 0 and 3 THEN '0-3'
    WHEN Tenure BETWEEN 4 and 7  THEN '4-7'
    WHEN Tenure BETWEEN 8 and 10 THEN '8-10'
  END AS Tenure_Category
FROM client
ORDER BY Tenure_Category; 
```
```text
# CustomerId	Surname	Geography	Gender	Age	Tenure	Balance	EstimatedSalary	Exited	Complain	Satisfaction Score	Point_Earned	IsActiveMember	Tenure_Category
15762091	Simpson	Germany	Male	46	1	170826.55	45041.32	0	0	2	689	0	0-3
15807457	Abernathy	Spain	Female	45	2	0	197789.83	1	1	4	603	0	0-3
15796167	Flores	France	Male	20	0	83459.86	146752.67	0	0	4	814	1	0-3
15777614	Webb	Spain	Male	27	2	172463.45	40315.27	0	0	3	704	1	0-3
15790763	Trujillo	France	Male	35	2	101257.16	118113.64	0	0	4	900	1	0-3

```
#### 10 Найти средний баланс, зарплату и рейтинг у клиентов в каждой категории.
```sql
SELECT 
  CASE 
    WHEN Tenure BETWEEN 0 and 3 THEN '0-3'
    WHEN Tenure BETWEEN 4 and 7  THEN '4-7'
    WHEN Tenure BETWEEN 8 and 10 THEN '8-10'
  END AS Tenure_Category, ROUND(AVG(Balance), 2) AS AVG_Баланс, ROUND(AVG(EstimatedSalary), 2) AS AVG_Зарплата, ROUND(AVG(Point_Earned), 2) AS AVG_Рейтинг
FROM client
GROUP BY Tenure_Category
ORDER BY AVG_Рейтинг;
```
```text

# Tenure_Category	AVG_Баланс	AVG_Зарплата	AVG_Рейтинг
8-10	76609.46	100760.24	600.95
0-3	77942.89	99242.02	607.79
4-7	75118.68	100440.37	608.97
```

#### 11 Найти кредитный рейтинг клиентов из Испании
```sql
SELECT CustomerId, CreditScore
FROM client c
	JOIN creditscore  cs ON c.CustomerId = cs.Customer
WHERE Geography = 'Spain'
ORDER BY CreditScore DESC
```
```text

# CustomerId	CreditScore
15630580	850
15614365	850
15635034	850
15665008	850
15625092	850
…
```
#### 12 Найти количество и процент клиентов использующие кредитные карты в каждой стране.
```sql
SELECT Geography, COUNT(Customer) AS Количество, (COUNT(Customer) / (SELECT COUNT(*) FROM creditcard)) * 100 AS Процент_от_общего_числа
FROM client c
	JOIN creditcard cc ON c.CustomerId = cc.Customer
GROUP BY Geography
ORDER BY Количество, Процент_от_общего_числа;
```
```text

# Geography	Количество	Процент_от_общего_числа
Spain	2477	24.7700
Germany	2508	25.0800
France	5014	50.1400
```
#### 13 Найти средний кредитный рейтинг у каждого типа кредитной карты клиентов из Германии.
```sql
SELECT CardType, AVG(creditScore) AS AVG_Рейтинг
FROM creditcard cc
	JOIN client c ON cc. Customer = c. CustomerId
    	JOIN creditscore USING(Customer)
WHERE Geography = 'Germany'
GROUP BY CardType
ORDER BY AVG_Рейтинг;
```
```text
# CardType	AVG_Рейтинг
DIAMOND	647.1744
SILVER	648.0850
PLATINUM	651.5806
GOLD	658.5997
```

#### 14 Какое среднее количество продуктов используют клиенты из возрастной группы 18-25 с ненулевым балансом в каждой стране.
```sql
SELECT AVG(NumOfProducts) AS AVG_Количество
FROM client c
	JOIN product p ON c. CustomerId = p. Customer
WHERE AGE BETWEEN 18 AND 25 AND Balance !=0
GROUP BY Geography 
ORDER BY AVG_Количество;
```
```text
# Geography	AVG_Количество
Spain	1.2875
France	1.2876
Germany	1.4895
```
#### 15 Понизить рейтинг клиента ушедших из банка до 0/
```sql
UPDATE client
SET Point_Earned = 0
WHERE Exited = 1;
```
### 16 Клиентам с кредитным рейтингом выше 850 и стажем более 7 лет выдали платиновые карты, обновить данные в таблице/
```sql
UPDATE Creditcard
SET CardType = 'Platinum'
WHERE Customer IN (
    SELECT client
    FROM Tenure
    WHERE Tenure > 7)
 AND
 Customer IN (
    SELECT Customer
    FROM creditscore
    WHERE creditscore  > 850);
###
UPDATE Creditcard cc
SET CardType = 'Platinum'
FROM Creditcard cc
INNER JOIN (
    SELECT client
    FROM Tenure
    WHERE Tenure > 7
) AS t ON cc.Customer = t.client
INNER JOIN (
    SELECT Customer
    FROM creditscore
    WHERE creditscore > 850
) AS cs ON cc.Customer = cs.Customer;
```
### 17 Выгрузить файл с клиентами ушедших из банка в Германии.
```sql
SELECT CustomerId, Surname, Gender, Point Earned 
FROM client 
WHERE Geography = 'Germany'
INTO OUTFILE '/var/lib/mysql-files/clints_out.csv'
```





