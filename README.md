# fa22-cs411-Q-team087-404
Team 404 Repository

## Procedure with 2 Advanced Queries

### Stored Procedure: CourseProcedure

```sql
delimiter //
CREATE PROCEDURE CourseProcedure(IN cid VARCHAR(20))
BEGIN
    DECLARE varCourseId VARCHAR(20);
    DECLARE varNumber INT;
    DECLARE varTitle VARCHAR(50);
    DECLARE varDeptId VARCHAR(10);
    DECLARE varEid VARCHAR(20);
    DECLARE varName VARCHAR(70);
    DECLARE avggpa REAL;
    DECLARE feeling VARCHAR(10);
    DECLARE level VARCHAR(30);
    DECLARE varA1 INT;
    DECLARE varA2 INT;
    DECLARE varA3 INT;
    DECLARE varB1 INT;
    DECLARE varB2 INT;
    DECLARE varB3 INT;
    DECLARE varC1 INT;
    DECLARE varC2 INT;
    DECLARE varC3 INT;
    DECLARE varD1 INT;
    DECLARE varD2 INT;
    DECLARE varD3 INT;
    DECLARE varF INT;
    DECLARE varW INT;
    DECLARE percent REAL DEFAULT 0;
    DECLARE exit_loop BOOLEAN DEFAULT FALSE;
    DECLARE cusCur CURSOR FOR (
        SELECT c.CourseId, c.Number, c.Title, c.DeptId, i.Eid, i.Name, num_each_grade.A1, num_each_grade.A2, num_each_grade.A3, 
               num_each_grade.B1, num_each_grade.B2, num_each_grade.B3, num_each_grade.C1, num_each_grade.C2, num_each_grade.C3, 
               num_each_grade.D1, num_each_grade.D2, num_each_grade.D3, num_each_grade.F, num_each_grade.W 
        FROM Courses AS c 
        NATURAL JOIN Course_Offerings AS co 
        NATURAL JOIN Instructors AS i 
        NATURAL JOIN (
            SELECT co.CourseId, i.Eid, 
                   SUM(co.A1) AS A1, SUM(co.A2) AS A2, SUM(co.A3) AS A3, 
                   SUM(co.B1) AS B1, SUM(co.B2) AS B2, SUM(co.B3) AS B3, 
                   SUM(co.C1) AS C1, SUM(co.C2) AS C2, SUM(co.C3) AS C3, 
                   SUM(co.D1) AS D1, SUM(co.D2) AS D2, SUM(co.D3) AS D3, 
                   SUM(co.F) AS F, SUM(co.W) AS W 
            FROM Course_Offerings AS co 
            NATURAL JOIN Instructors AS i 
            GROUP BY co.CourseId, i.Eid
        ) AS num_each_grade 
        WHERE c.CourseId = cid
    );
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET exit_loop = TRUE;

    DROP TABLE IF EXISTS NewTable;
    CREATE TABLE NewTable(
        CourseId VARCHAR(20),
        Number INT,
        Title VARCHAR(50),
        DeptId VARCHAR(10),
        Eid VARCHAR(20),
        Name VARCHAR(70),
        gpa REAL,
        feel VARCHAR(10),
        diff VARCHAR(30)
    );

    OPEN cusCur;
    cloop: LOOP
        FETCH cusCur INTO varCourseId, varNumber, varTitle, varDeptId, varEid, varName, 
                           varA1, varA2, varA3, varB1, varB2, varB3, 
                           varC1, varC2, varC3, varD1, varD2, varD3, varF, varW;
        IF exit_loop THEN
            LEAVE cloop;
        END IF;

        SET avggpa = ((4 * varA1) + (3.66 * varA2) + (3.33 * varA3) + 
                      (3 * varB1) + (2.66 * varB2) + (2.33 * varB3) + 
                      (2 * varC1) + (1.66 * varC2) + (1.33 * varC3) + 
                      (1 * varD1) + (0.66 * varD2) + (0.33 * varD3) + 
                      (0 * varF)) / 
                     (varA1 + varA2 + varA3 + varB1 + varB2 + varB3 + 
                      varC1 + varC2 + varC3 + varD1 + varD2 + varD3 + varF);

        IF (avggpa > 3.5) THEN
            SET feeling = ":D";
        ELSEIF (avggpa > 3) THEN
            SET feeling = ":)";
        ELSEIF (avggpa > 2.5) THEN
            SET feeling = ":/";
        ELSEIF (avggpa > 2) THEN
            SET feeling = ":(";
        ELSE
            SET feeling = "D:";
        END IF;

        SELECT output.percentage * 100 AS percentage INTO percent 
        FROM (
            SELECT CourseId, SUM(A1 + A2 + A3) AS A, 
                   SUM(A1 + A2 + A3 + B1 + B2 + B3 + C1 + C2 + C3 + D1 + D2 + D3 + F) AS total, 
                   SUM(A1 + A2 + A3) / SUM(A1 + A2 + A3 + B1 + B2 + B3 + C1 + C2 + C3 + D1 + D2 + D3 + F) AS percentage 
            FROM Course_Offerings 
            GROUP BY CourseId
        ) AS output 
        JOIN Courses ON output.CourseId = Courses.CourseId 
        WHERE output.CourseId = cid;

        IF (percent > 80) THEN
            SET level = "Easy";
        ELSEIF (percent > 50) THEN
            SET level = "Moderate";
        ELSE
            SET level = "Hard";
        END IF;

        INSERT INTO NewTable VALUES (varCourseId, varNumber, varTitle, varDeptId, varEid, varName, avggpa, feeling, level);
    END LOOP cloop;
    CLOSE cusCur;
    SELECT DeptId, Number, CourseId, Title, Name, gpa, feel 
    FROM NewTable 
    WHERE CourseId = cid 
    ORDER BY DeptId, Number, CourseId, Title, Name;
END;
```
### Trigger:
```sql
delimiter //
BEGIN
    SET @dif_rating = (SELECT AVG(Difficulty_rating) FROM Reviews WHERE CourseId = NEW.CourseId);
    SET @pra_rating = (SELECT AVG(Practical_rating) FROM Reviews WHERE CourseId = NEW.CourseId);
    SET @workload = (SELECT AVG(Workload_hours) FROM Reviews WHERE CourseId = NEW.CourseId);
    SET @crs_rating = (SELECT AVG(Course_rating) FROM Reviews WHERE CourseId = NEW.CourseId);
    
    IF NEW.Difficulty_rating > 0 THEN
        UPDATE Courses SET Difficulty_rating = @dif_rating WHERE CourseId = NEW.CourseId;
    END IF;
    IF NEW.Practical_rating > 0 THEN
        UPDATE Courses SET Practical_rating = @pra_rating WHERE CourseId = NEW.CourseId;
    END IF;
    IF NEW.Workload_hours > 0 THEN
        UPDATE Courses SET Workload_hours = @workload WHERE CourseId = NEW.CourseId;
    END IF;
    IF NEW.Course_rating > 0 THEN
        UPDATE Courses SET Course_rating = @crs_rating WHERE CourseId = NEW.CourseId;
    END IF;
END;
```
