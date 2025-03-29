# Functions

## 1. Calculating the average grade for a student
```sql
CREATE FUNCTION CalculateAverageGradeForStudent
(
  @StudentID int,
  @StudiesID int
)
RETURNS DECIMAL(5, 2)
AS
BEGIN
  DECLARE @AverageGrade DECIMAL(5, 2);

  SELECT @AverageGrade = AVG(Mark)
  FROM Exams
  WHERE StudentID = @StudentID and StudiesID = @StudiesID;
  
  RETURN ISNULL(@AverageGrade, 0); -- Jeśli nie ma ocen, zwraca 0
END;
```

## 2. Number of students present at the class
```sql
CREATE FUNCTION GetClassAttendanceCount
(
   @ClassID INT
)
RETURNS INT
AS
BEGIN
   IF EXISTS (SELECT 1 FROM Attendance WHERE ClassId = @ClassID)
       BEGIN
       DECLARE @AttendanceCount INT;

       SELECT @AttendanceCount = COUNT(*)
       FROM Attendance
       WHERE ClassID = @ClassID AND Present = 1

       RETURN @AttendanceCount;
   END
   RETURN 0
END;
```

## 3. Calculating the number of days remaining until the completion of the educational program
```sql
CREATE FUNCTION DaysRemainingInProgram
(
   @ProgramID INT
)
RETURNS INT
AS
BEGIN
   DECLARE @DaysRemaining INT;

   SELECT @DaysRemaining = DATEDIFF(DAY, GETDATE(), ProgramEnd)
   FROM EducationalPrograms
   WHERE ProgramID = @ProgramID;

   IF @DaysRemaining < 0 -- Jeżeli program już się zakończył, zwróć 0
       SET @DaysRemaining = 0;

   RETURN @DaysRemaining;
END;
```

## 4. Calculating the total amount for all programs in a given order

```sql
CREATE FUNCTION CalculateFullPriceForOrder
(
    @OrderID int 
)
RETURNS MONEY
AS
BEGIN
    DECLARE @fullprice MONEY;
    SELECT @fullprice = 
    SUM(ISNULL(EP.ProgramPrice,0)) + SUM(ISNULL(C.ClassPrice,0))
    FROM Orders O
    LEFT JOIN RegisteredPrograms RP ON RP.OrderID = O.OrderID
    LEFT JOIN RegisteredClasses RC ON O.OrderID = RC.OrderID
    LEFT JOIN Classes C ON RC.ClassID = C.ClassID
    LEFT JOIN EducationalPrograms EP ON EP.ProgramID = RP.ProgramID
    GROUP BY O.OrderID
    HAVING O.OrderId = @OrderID

    RETURN ISNULL(@fullprice, 0)

END;
```

## 5. Calculating the sum of registration fees for programs in a given order

```sql
CREATE FUNCTION CalculateEntryPriceForOrder
(
    @OrderID int
)
RETURNS MONEY
AS
BEGIN
    DECLARE @entryprice MONEY;
    SELECT @entryprice = 
    SUM(ISNULL(S.EntryFee,0)) + SUM(ISNULL(CS.Advance,0)) + SUM(ISNULL(C.ClassPrice,0))
    FROM Orders O
    LEFT JOIN RegisteredPrograms RP ON RP.OrderID = O.OrderID
    LEFT JOIN RegisteredClasses RC ON O.OrderID = RC.OrderID
    LEFT JOIN Classes C ON RC.ClassID = C.ClassID
    LEFT JOIN EducationalPrograms EP ON EP.ProgramID = RP.ProgramID
    LEFT JOIN Studies S ON EP.StudiesID = S.StudiesID
    LEFT JOIN Courses CS ON CS.CourseID = EP.CourseID
    LEFT JOIN Webinars W ON W.WebinarID = EP.WebinarID
    GROUP BY O.OrderID
    HAVING O.OrderID = @OrderID

    RETURN ISNULL(@entryprice, 0)

END;
```
<div style="page-break-after: always;"></div>

## 6. Calculating the total amount spent by a student on educational programs for a specific time period
```sql
CREATE FUNCTION CalculateTotalPaymentsForStudent
(
  @StudentID int,
  @startDate date,
  @endDate date
)
RETURNS MONEY
AS
BEGIN
  DECLARE @TotalPayments MONEY;

  SELECT @TotalPayments = SUM(Amount)
  FROM Payments
  WHERE OrderID IN (SELECT OrderID FROM Orders WHERE StudentID = @StudentID) AND status = 1 AND date between @startDate and @endDate;
  
  RETURN ISNULL(@TotalPayments, 0);
END;
```

## 7. Displaying the class schedule for a specific day for a specific student
```sql
CREATE FUNCTION ScheduleForStudent(@StudentID int, @day DATE)
    RETURNS TABLE
        AS
        RETURN
        SELECT Student, ModuleName, SubjectName, Teacher, StartTime, EndTime, RoomNumber
        FROM OfflineParticipantsList as ofp
        WHERE @StudentID = ofp.StudentID and @day = cast(StartTime as date) and Access = 'true'
```

## 8. Calculating the number of programs ordered by students in a given year
```sql
CREATE FUNCTION OrdersProgramsAmount(@year int)
   RETURNS INT
   AS
   BEGIN
       DECLARE @Amount INT;
       SET @Amount = (
       select count(*) from orders
       inner join RegisteredPrograms as rp
           on orders.OrderID = rp.OrderID
           where year(cast (OrderDate as DATE))  = @year)
       RETURN @Amount
   END
```

## 9. Displaying currently ongoing synchronous online classes
```sql
CREATE FUNCTION LiveOnlineSynchClasses()
   RETURNS TABLE
       AS
       RETURN
       SELECT c.StartTime, c.EndTime, oc.Link from OnlineClasses as oc
           INNER JOIN Classes as c on oc.ClassID = c.ClassID
           WHERE oc.Synch = 'true' AND GETDATE() between c.StartTime AND c.EndTime
```
<div style="page-break-after: always;"></div>

## 10. Calculating the average grade for individual sessions (only if a grade has been assigned to each participant of the session)
```sql
CREATE FUNCTION AverageMarkOnClass(@ClassID int)
   RETURNS INT
   BEGIN
       DECLARE @Average int
       IF NOT EXISTS (SELECT MARK FROM Attendance WHERE MARK is null and @ClassID = Attendance.ClassID)
           SET @Average = (select AVG(MARK) from Attendance where @ClassID = Attendance.ClassID)
       ELSE set @Average = null
       RETURN @Average
   END
```

## 11. Function checking the minimum possible number of participants for a session within a study module (if the session is added to the studies)
```sql
CREATE FUNCTION CalculateMinClassParticipantsForStudies(@ModuleID int)
RETURNS INT
AS
BEGIN
  DECLARE @MinParticipants INT;
  SET @MinParticipants = COALESCE((select s.MaxParticipants
   from Modules as m
       inner join EducationalPrograms as ep
           on m.ProgramID = ep.ProgramID
       inner join Studies as s
           on ep.StudiesID = s.StudiesID
       where ModuleID = @ModuleID), 0)
  RETURN @MinParticipants;
END;
```

## 12. Displaying the list of people enrolled in a specific offline event within a study program
```sql
CREATE FUNCTION AllClassParticipants(@ClassID int)
    RETURNS TABLE
        AS
        RETURN
        with t as (
        select c.ClassID, s.StudiesID
        from Classes as c
            inner join Modules as m
                on m.ModuleID = c.ModuleID
            inner join EducationalPrograms as ep
                on m.ProgramID = ep.ProgramID
            inner join Studies as s
                on ep.StudiesID = s.StudiesID
        )

        select StudentID, Student, StudiesID, Access
            from OfflineParticipantsList
                inner join t
                    on t.ClassID = OfflineParticipantsList.ClassID and t.ClassID = @ClassID
        UNION
        select StudentID, Student, StudiesID, Access
            from StudentsOuterClasses
                inner join t
                    on t.ClassID = StudentsOuterClasses.ClassID and t.ClassID = @ClassID
```

<div style="page-break-after: always;"></div>

## 13. Displaying the list of people enrolled in a specific program
```sql
CREATE FUNCTION AllProgramParticipants(@ProgramID int)
    RETURNS TABLE
        AS
        RETURN
        select distinct StudentID, Student, Access
        from StudentsPrograms as sp
            inner join EducationalPrograms as ep
                on ep.ProgramID = sp.ProgramID
        where ep.ProgramID = @ProgramID
```

## 14. Displaying the list of studies a student is enrolled in and has access to. This function is used in procedure #18 and trigger #2
```sql
CREATE FUNCTION StudentStudies(@StudentID int)
    RETURNS TABLE
        AS
        RETURN
        select StudentID, Student, RegisteredProgramID, StudiesID, sp.ProgramName, sp.ProgramStart, sp.ProgramEnd, sp.Passed
        from StudentsPrograms as sp
            inner join EducationalPrograms as ep
                on ep.ProgramID = sp.ProgramID
        where @StudentID = sp.StudentID and StudiesID is not null and access = 'true'
```

## 15. Displaying the list of webinars a student is enrolled in and has access to
```sql
CREATE FUNCTION StudentWebinars(@StudentID int)
    RETURNS TABLE
        AS
        RETURN
        select StudentID, Student, RegisteredProgramID, WebinarID, sp.ProgramName, sp.ProgramStart, sp.ProgramEnd, sp.Passed
        from StudentsPrograms as sp
            inner join EducationalPrograms as ep
                on ep.ProgramID = sp.ProgramID
        where @StudentID = sp.StudentID and WebinarID is not null and access = 'true'
```

## 16. Displaying the list of courses a student is enrolled in and has access to
```sql
CREATE FUNCTION StudentCourses(@StudentID int)
    RETURNS TABLE
        AS
        RETURN
        select StudentID, Student, RegisteredProgramID, CourseID, sp.ProgramName, sp.ProgramStart, sp.ProgramEnd, sp.Passed
        from StudentsPrograms as sp
            inner join EducationalPrograms as ep
                on ep.ProgramID = sp.ProgramID
        where @StudentID = sp.StudentID and CourseID is not null and access = 'true'
```