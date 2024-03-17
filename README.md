# PromAtSQL

Задача 1:
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
    GO
    
    CREATE PROCEDURE PrintEvenNumbers
    AS
    BEGIN
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
    END

Пример использования:
`SELECT dbo.ConvertSecondsToDHMS(123456) AS FormattedTime
EXEC PrintEvenNumbers`
