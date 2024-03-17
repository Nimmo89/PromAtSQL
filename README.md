# PromAtSQL

Задача 1:
Создайте процедуру ИЛИ функцию, которая принимает кол-во сек и формат их в кол-во дней, часов, минут и секунд.
* Пример: 123456 ->'1 days 10 hours 17 minutes 36 seconds '
---
    CREATE FUNCTION ConvertSecondsToDHMS (@totalSeconds INT)
    RETURNS NVARCHAR(100)
    AS
    BEGIN
        DECLARE @days INT, @hours INT, @minutes INT, @seconds INT
    
        SET @days = @totalSeconds / (24 * 3600)
        SET @totalSeconds = @totalSeconds % (24 * 3600)
        
        SET @hours = @totalSeconds / 3600
        SET @totalSeconds = @totalSeconds % 3600
        
        SET @minutes = @totalSeconds / 60
        SET @seconds = @totalSeconds % 60
        
        RETURN CONCAT(@days, ' days ', @hours, ' hours ', @minutes, ' minutes ', @seconds, ' seconds')
    END

Пример использования:
`SELECT dbo.ConvertSecondsToDHMS(123456) AS Result`

---
Задача 2:
Выведите только четные числа от 1 до 10 (Через цикл).
* Пример: 2,4,6,8,10
---
    DECLARE @counter INT
    SET @counter = 1
    
    WHILE @counter <= 10
    BEGIN
        IF @counter % 2 = 0
        BEGIN
            PRINT @counter
        END
        SET @counter = @counter + 1
    END

---
Задача 3:
Создать процедуру, которая решает следующую задачу
* Выбрать для одного пользователя 5 пользователей в случайной комбинации, которые удовлетворяют хотя бы одному критерию:
    * из одного города
    * состоят в одной группе
    * друзья друзей


---
    CREATE PROCEDURE SelectRandomUsers
        @userId INT
    AS
    BEGIN
        DECLARE @cityUsers TABLE (UserId INT)
        DECLARE @groupUsers TABLE (UserId INT)
        DECLARE @friendsOfFriends TABLE (UserId INT)
    
        -- Выбор пользователей из одного города
        INSERT INTO @cityUsers (UserId)
        SELECT TOP 5 UserId
        FROM Users
        WHERE City = (SELECT City FROM Users WHERE UserId = @userId)
        AND UserId != @userId
        ORDER BY NEWID()
    
        -- Выбор пользователей из одной группы
        INSERT INTO @groupUsers (UserId)
        SELECT TOP 5 UserId
        FROM Users
        WHERE GroupId = (SELECT GroupId FROM Users WHERE UserId = @userId)
        AND UserId != @userId
        ORDER BY NEWID()
    
        -- Выбор друзей друзей
        INSERT INTO @friendsOfFriends (UserId)
        SELECT TOP 5 UserId
        FROM Users
        WHERE UserId IN (
            SELECT DISTINCT U.UserId
            FROM Users U
            INNER JOIN Friends F ON U.UserId = F.FriendId
            WHERE F.UserId IN (
                SELECT FriendId
                FROM Friends
                WHERE UserId = @userId
            )
            AND U.UserId != @userId
        )
        ORDER BY NEWID()
    
        SELECT 'Users from the same city:', * FROM @cityUsers
        SELECT 'Users from the same group:', * FROM @groupUsers
        SELECT 'Friends of friends:', * FROM @friendsOfFriends
    END

Пример использования:
`EXEC SelectRandomUsers @userId = 1`

---
Задача 4:
Создать функцию, вычисляющей коэффициент популярности пользователя 
* по количеству друзей
---
    CREATE FUNCTION CalculatePopularityCoefficient (@userId INT)
    RETURNS INT
    AS
    BEGIN
        DECLARE @friendCount INT
    
        SELECT @friendCount = COUNT(*)
        FROM Friends
        WHERE UserId = @userId
    
        RETURN @friendCount
    END

Пример использования:
`SELECT dbo.CalculatePopularityCoefficient(1) AS PopularityCoefficient`

---
Задача 5:
Создайте хранимую функцию hello(), которая будет возвращать приветствие, в зависимости от текущего времени суток. 
* Пример: С 6:00 до 12:00 функция должна возвращать фразу "Доброе утро", с 12:00 до 18:00 функция должна возвращать фразу "Добрый день", с 18:00 до 00:00 — "Добрый вечер", с 00:00 до 6:00 — "Доброй ночи".
---
    CREATE FUNCTION hello()
    RETURNS NVARCHAR(50)
    AS
    BEGIN
        DECLARE @currentTime AS TIME
        SET @currentTime = CONVERT(TIME, GETDATE())
    
        IF @currentTime >= '06:00' AND @currentTime < '12:00'
            RETURN 'Доброе утро'
        ELSE IF @currentTime >= '12:00' AND @currentTime < '18:00'
            RETURN 'Добрый день'
        ELSE IF @currentTime >= '18:00' AND @currentTime < '00:00'
            RETURN 'Добрый вечер'
        ELSE
            RETURN 'Доброй ночи'
    END

Пример использовани:
`SELECT dbo.hello() AS Greeting`

---
Задача 6:
Создайте таблицу logs типа Archive. Пусть при каждом создании записи в таблицах users, communities и messages в таблицу logs помещается время и дата создания записи, название таблицы, идентификатор первичного ключа. 
* Используя Триггеры
---
    -- Создание таблицы logs типа Archive
    CREATE TABLE logs (
        log_id INT IDENTITY(1,1) PRIMARY KEY,
        table_name NVARCHAR(50),
        primary_key_id INT,
        created_at DATETIME DEFAULT GETDATE()
    ) ON [PRIMARY]
    GO
    
    -- Создание триггера для таблицы users
    CREATE TRIGGER trg_users_log
    ON users
    AFTER INSERT
    AS
    BEGIN
        INSERT INTO logs (table_name, primary_key_id)
        SELECT 'users', inserted.user_id
        FROM inserted
    END
    GO
    
    -- Создание триггера для таблицы communities
    CREATE TRIGGER trg_communities_log
    ON communities
    AFTER INSERT
    AS
    BEGIN
        INSERT INTO logs (table_name, primary_key_id)
        SELECT 'communities', inserted.community_id
        FROM inserted
    END
    GO
    
    -- Создание триггера для таблицы messages
    CREATE TRIGGER trg_messages_log
    ON messages
    AFTER INSERT
    AS
    BEGIN
        INSERT INTO logs (table_name, primary_key_id)
        SELECT 'messages', inserted.message_id
        FROM inserted
    END
    GO
