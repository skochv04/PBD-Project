# Procedures

## 1. Adding a New Student
```sql
CREATE PROCEDURE AddStudent(
   @firstName VARCHAR(20),
   @lastName VARCHAR(20),
   @countryID INT,
   @email VARCHAR(40)
)
AS
BEGIN
   BEGIN TRY
       SET NOCOUNT ON;
       IF EXISTS(SELECT * FROM Students WHERE Email = @email)
            THROW 52034, N'Email already in use', 1;
       IF NOT EXISTS(SELECT * FROM Countries WHERE CountryID = @CountryID)
            THROW 52034, N'Country does not exist in the Countries table', 1;
       INSERT INTO Students (FirstName, LastName, CountryID, Email)
       VALUES (@firstName, @lastName, @countryID, @email);
       PRINT 'Student added successfully.';
   END TRY
   BEGIN CATCH
       DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
       THROW 52034, @Message, 1;
   END CATCH
END
```
## 2. Deleting a Student's Data
```sql
ALTER PROCEDURE DeleteStudent(@studentID INT)
AS
BEGIN
   BEGIN TRY
       IF NOT EXISTS(
               SELECT *
               FROM Students
               WHERE StudentID = @studentID
           )
           BEGIN
               THROW 52000, N'There is no student with given ID', 1;
           END
           DECLARE @an NVARCHAR(10) = 'xxxxxxxx'
           UPDATE Students
               SET FirstName   = @an,
               LastName    = @an,
               Email     = @an
           WHERE StudentID = @studentID
   END TRY
   BEGIN CATCH
       DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
       THROW 52000, @msg, 1;
   END CATCH
END
```


<div style="page-break-after: always;"></div>

## 3. Adding a New Course
```sql
CREATE PROCEDURE AddCourse
   @Place varchar(40),
   @Advance money,
   @ProgramName varchar(100),
   @Language varchar(20),
   @ProgramStart date,
   @ProgramEnd date,
   @ProgramPrice money,
   @LecturerID int,
   @TranslatorID int
AS
BEGIN
BEGIN TRY
   BEGIN TRANSACTION
   SET NOCOUNT ON;

   IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @LecturerID)
   BEGIN
       THROW 50000, 'TeacherID does not exist in the Teachers table.', 1;
   END;
   IF NOT EXISTS (SELECT 1 FROM Translators WHERE TranslatorID = @TranslatorID)
   BEGIN
       THROW 50000, 'TranslatorID does not exist in the Translators table.', 1;
   END;

   INSERT INTO Courses (Place, Advance)
   VALUES (@Place, @Advance);

   INSERT INTO EducationalPrograms (ProgramName, CourseID, Language, ProgramStart, ProgramEnd, ProgramPrice, LecturerID, TranslatorID)
   VALUES (@NewProgramID, @ProgramName, @NewCourseID, @Language, @ProgramStart, @ProgramEnd, @ProgramPrice, @LecturerID, @TranslatorID);

   COMMIT TRANSACTION
   PRINT 'Course added successfully.';
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION
    DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
    THROW 52011, @Message, 1;
END CATCH
END;
```

## 4. Updating Student Data (only email or country)
```sql
CREATE PROCEDURE ChangeStudentData(
  @studentID int,
  @countryID int = NULL,
  @email varchar(40) = NULL)
AS
BEGIN
   SET NOCOUNT ON;
   IF @countryID IS NOT NULL
   BEGIN
       IF @countryID in (select CountryID from Countries)
           UPDATE Students SET CountryID = @countryID WHERE StudentID = @studentID
   END
   IF @email IS NOT NULL
   BEGIN
       UPDATE Students SET Email = @Email WHERE StudentID = @studentID
       PRINT 'Student data changed successfully.';
   END
END
```

## 5. Adding a new teacher
```sql
CREATE PROCEDURE AddTeacher(
  @firstName VARCHAR(20),
  @lastName VARCHAR(20),
  @countryID INT
)
AS
BEGIN
  BEGIN TRY
    IF NOT EXISTS(SELECT * FROM Countries WHERE CountryID = @CountryID)
        THROW 52034, N'Country does not exist in the Countries table', 1;
    INSERT INTO Teachers (FirstName, LastName, CountryID)
    VALUES (@firstName, @lastName, @countryID)
    PRINT 'Teacher added successfully.';
  END TRY
  BEGIN CATCH
        DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
        THROW 52011, @Message, 1;
  END CATCH
END
```

## 6. Marking the make-up of student absences for classes
```sql
CREATE PROCEDURE redoAttendance (
    @classID int, 
    @studentID int
    )
AS
BEGIN
   SET NOCOUNT ON
   BEGIN TRY
       IF NOT EXISTS
           (
            select ClassID, ParticipantID
               from Attendance
           WHERE ClassID = @classID and ParticipantID = @studentID
           )
       BEGIN
           THROW 52000, N'The student was not registered for the class with the given ID', 1
       END
       IF EXISTS
           (
           select ClassID, ParticipantID
               from Attendance
           WHERE ClassID = @classID and ParticipantID = @studentID and Redone = 1
           )
       BEGIN
           THROW 52000, N'Attendance had already been made up earlier', 1
       END
       UPDATE Attendance
       SET Redone = 1
       where ClassID = @classID and ParticipantID = @studentID
       PRINT 'Attendance was successfully set as redone!'
   END TRY
   BEGIN CATCH
       DECLARE @Message NVARCHAR(1000)= 'Error when setting attendance as made up: ' + ERROR_MESSAGE();
       THROW 52011, @Message, 1
   END CATCH
END
```

<div style="page-break-after: always;"></div>

## 7. Adding an order

```sql
CREATE PROCEDURE AddOrder(
   @Studentid INT
)
AS
BEGIN
  BEGIN TRY
      IF NOT EXISTS(SELECT * FROM Students WHERE StudentID = @Studentid)
      BEGIN
          THROW 52011, N'There is no student with such id', 1;
      END
      BEGIN
          INSERT INTO Orders (StudentID, OrderDate)
          VALUES (@Studentid, GETDATE());

          PRINT 'Order added successfully.';
       END
  END TRY
  BEGIN CATCH
      DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
      THROW 52011, @Message, 1;
  END CATCH
END
```
## 8. Adding classes to an order
```sql
CREATE PROCEDURE RegisterClass(
   @OrderID INT,
   @ClassID INT
)
AS
BEGIN
  BEGIN TRY
       IF NOT EXISTS(SELECT * FROM Orders WHERE OrderID = @OrderID)
        THROW 52313, N'There is no Order with such id', 1;


       IF NOT EXISTS(SELECT * FROM Classes WHERE ClassID = @ClassID)
        THROW 52313, N'There is no Class with such id', 1;


       IF EXISTS (SELECT * FROM RegisteredClasses WHERE OrderID = @OrderID and ClassID = @ClassID)
        THROW 52313, N'Student has already registered for this class', 1;

       INSERT INTO RegisteredClasses (OrderID, ClassID)
       VALUES (@OrderID, @ClassID);

       PRINT 'Class was added successfully!'
  END TRY
  BEGIN CATCH
      DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
      THROW 52011, @Message, 1;
  END CATCH
END
```

<div style="page-break-after: always;"></div>

## 9. Adding an educational program to an order
```sql
CREATE PROCEDURE RegisterProgram(
 @OrderID INT,
 @ProgramID INT,
 @Passed AS bit = FALSE,
 @CertificateLink AS varchar(255) = NULL
)
AS
BEGIN
BEGIN TRY
     IF NOT EXISTS(SELECT * FROM Orders WHERE OrderID = @OrderID)
      THROW 52313, N'There is no Order with such id', 1;

     IF NOT EXISTS(SELECT * FROM EducationalPrograms WHERE ProgramID = @ProgramID)
      THROW 52313, N'There is no EducationalProgram with such id', 1;

     IF EXISTS (SELECT * FROM RegisteredPrograms WHERE OrderID = @OrderID and ProgramID = @ProgramID)
       THROW 52313, N'Student has already registered for this educational program', 1;

     INSERT INTO RegisteredPrograms (OrderID, ProgramID, Passed, CertificateLink)
     VALUES (@OrderID, @ProgramID, @Passed, @CertificateLink);
     PRINT 'Program was added successfully!'
END TRY
BEGIN CATCH
    DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
    THROW 52011, @Message, 1;
END CATCH
END;
```

<div style="page-break-after: always;"></div>

## 10. Adding a new online class
```sql
CREATE PROCEDURE AddOnlineClass
 @Link varchar(255),
 @Synch bit,
 @TeacherID int,
 @SubjectID int,
 @StartTime datetime,
 @EndTime datetime,
 @ClassPrice money = NULL,
 @ModuleID int = NULL,
 @PractiseID int = NULL,
 @NewClassID int OUTPUT

AS
BEGIN
 SET NOCOUNT ON;
 BEGIN TRY
     BEGIN TRANSACTION
     IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @TeacherID)
     BEGIN
         THROW 50000, 'TeacherID does not exist in the Teachers table.', 1;
     END;

     IF NOT EXISTS (SELECT 1 FROM Subjects WHERE SubjectID = @SubjectID)
     BEGIN
         THROW 50000, 'SubjectID does not exist in the Subjects table.', 1;
     END;

     IF (@ModuleID IS NOT NULL AND @PractiseID IS NOT NULL)
     BEGIN
         THROW 50000, 'Can`t define both ModuleID and PractiseID', 1;
     END;

     IF @ModuleID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Modules WHERE ModuleID = @ModuleID)
     BEGIN
         THROW 50000, 'ModuleID does not exist in the Modules table.', 1;
     END;

     IF @PractiseID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Practises WHERE PractiseID = @PractiseID)
     BEGIN
         THROW 50000, 'PractiseID does not exist in the Practises table.', 1;
     END;

     INSERT INTO Classes (TeacherID, SubjectID, StartTime, EndTime, ClassPrice)
     VALUES (@TeacherID, @SubjectID, @StartTime, @EndTime, @ClassPrice);

     INSERT INTO OnlineClasses (Link, Synch)
     VALUES (@Link, @Synch)

     IF @ModuleID IS NOT NULL
     BEGIN
         UPDATE Classes SET ModuleID = @ModuleID WHERE ClassID = @NewClassID
     END;

     IF @PractiseID IS NOT NULL
     BEGIN
         UPDATE Classes SET PractiseID = @PractiseID WHERE ClassID = @NewClassID
     END;
     COMMIT TRANSACTION
     PRINT 'OnlineClass added successfully.';
 END TRY
 BEGIN CATCH
     ROLLBACK TRANSACTION
     DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
     THROW 52000, @msg, 1;
 END CATCH
END;
```

## 11. Adding a new in-person class
```sql
CREATE PROCEDURE AddOfflineClass
  @RoomNumber int,
  @MaxParticipants int,
  @TeacherID int,
  @SubjectID int,
  @StartTime datetime,
  @EndTime datetime,
  @ClassPrice money = NULL,
  @ModuleID int,
  @PractiseID int = NULL,
  @NewClassID int OUTPUT
AS
BEGIN
  SET NOCOUNT ON;
  BEGIN TRY
      BEGIN TRANSACTION
      IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @TeacherID)
      BEGIN
          THROW 50000, 'TeacherID does not exist in the Teachers table.', 1;
      END;

      IF NOT EXISTS (SELECT 1 FROM Subjects WHERE SubjectID = @SubjectID)
      BEGIN
          THROW 50000, 'SubjectID does not exist in the Subjects table.', 1;
      END;

      IF @ModuleID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Modules WHERE ModuleID = @ModuleID)
      BEGIN
          THROW 50000, 'ModuleID does not exist in the Modules table.', 1;
      END;

      IF @PractiseID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Practises WHERE PractiseID = @PractiseID)
      BEGIN
          THROW 50000, 'PractiseID does not exist in the Practises table.', 1;
      END;

      -- Sprawdzenie, czy w ramach tych studiów można dodać zajęcia z taką maksymalną ilością miejsc (musi ona być większa bądż równa od maksymalnej ilości miejsc dla studiów, żeby wszystkie studenty mogli się zmieszcić, a dodatkowo mogą pojawić się miejsca dla osób "z zewnątrz"

      DECLARE @MinParticipants INT;
      SET @MinParticipants = dbo.CalculateMinClassParticipantsForStudies (@ModuleID);

      IF @MaxParticipants < @MinParticipants
      BEGIN
          THROW 50000, 'MaxParticipants of class should be greater or equal to MaxParticipants of studies within which classes take place.', 1;
      END;
      INSERT INTO Classes (TeacherID, SubjectID, StartTime, EndTime, ClassPrice, ModuleID)
      VALUES (@TeacherID, @SubjectID, @StartTime, @EndTime, @ClassPrice, @ModuleID);

      INSERT INTO OfflineClasses (RoomNumber, MaxParticipants)
      VALUES (@RoomNumber, @MaxParticipants)
      IF @PractiseID IS NOT NULL
      BEGIN
          UPDATE Classes SET PractiseID = @PractiseID WHERE ClassID = @NewClassID
      END;
      COMMIT TRANSACTION
      PRINT 'OfflineClass added successfully.';
  END TRY
  BEGIN CATCH
      ROLLBACK TRANSACTION
      DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
      THROW 52000, @msg, 1;
  END CATCH
END;
```

## 12. Adding a new webinar
```sql
CREATE PROCEDURE AddWebinar
  @ProgramName varchar(100),
  @Language varchar(20),
  @ProgramStart date,
  @ProgramEnd date,
  @ProgramPrice money,
  @LecturerID int,
  @TranslatorID int = NULL,


  @Link varchar(255),
  @Synch bit,
  @SubjectID int,
  @StartTime datetime,
  @EndTime datetime,
  @ClassPrice money = NULL,
  @ModuleID int = NULL,
  @PractiseID int = NULL


AS
BEGIN
  SET NOCOUNT ON;
  BEGIN TRY
      BEGIN TRANSACTION

      DECLARE @NewClassID int;
      DECLARE @NewWebinarID int;

      IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @LecturerID)
      BEGIN
          THROW 50000, 'LecturerID does not exist in the Teachers table.', 1;
      END;

      IF @TranslatorID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Translators WHERE TranslatorID = @TranslatorID)
      BEGIN
          THROW 50000, 'TranslatorID does not exist in the Translators table.', 1;
      END;

      EXEC AddOnlineClass @Link, @Synch, @LecturerID, @SubjectID, @StartTime, @EndTime, @ClassPrice, @ModuleID, @PractiseID, @NewClassID OUTPUT

      INSERT INTO Webinars (ClassID)
      VALUES (@NewClassID);

      SELECT @NewWebinarID = ISNULL(MAX(WebinarID), 0)

      INSERT INTO EducationalPrograms (ProgramName, WebinarID, Language, ProgramStart, ProgramEnd, ProgramPrice, LecturerID, TranslatorID)
      VALUES (@ProgramName, @NewWebinarID, @Language, @ProgramStart, @ProgramEnd, @ProgramPrice, @LecturerID, @TranslatorID);

      COMMIT TRANSACTION
      PRINT 'Webinar added successfully.';
  END TRY
  BEGIN CATCH
      ROLLBACK TRANSACTION
      DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
      THROW 52000, @msg, 1;
  END CATCH
END;

```
<div style="page-break-after: always;"></div>


## 13. Adding a new course
```sql
CREATE PROCEDURE AddCourse
  @ProgramName varchar(100),
  @Language varchar(20),
  @ProgramStart date,
  @ProgramEnd date,
  @ProgramPrice money,
  @LecturerID int,
  @TranslatorID int = NULL,
  @Place varchar(20),
  @Advance money


AS
BEGIN
  SET NOCOUNT ON;
  BEGIN TRY
    
      BEGIN TRANSACTION

      DECLARE @NewCourseID int;

      IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @LecturerID)
      BEGIN
          THROW 50000, 'LecturerID does not exist in the Teachers table.', 1;
      END;


      IF @TranslatorID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Translators WHERE TranslatorID = @TranslatorID)
      BEGIN
          THROW 50000, 'TranslatorID does not exist in the Translators table.', 1;
      END;


      INSERT INTO Courses (Place, Advance)
      VALUES (@Place, @Advance);

      SELECT @NewCourseID = ISNULL(MAX(CourseID), 0)

      INSERT INTO EducationalPrograms (ProgramName, CourseID, Language, ProgramStart, ProgramEnd, ProgramPrice, LecturerID, TranslatorID)
      VALUES (@ProgramName, @NewCourseID, @Language, @ProgramStart, @ProgramEnd, @ProgramPrice, @LecturerID, @TranslatorID);

      COMMIT TRANSACTION
      PRINT 'Course added successfully.';
  END TRY
  BEGIN CATCH
      ROLLBACK TRANSACTION
      DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
      THROW 52000, @msg, 1;
  END CATCH
END;

```
<div style="page-break-after: always;"></div>


## 14. Adding new studies
```sql
CREATE PROCEDURE AddStudies
  @ProgramName varchar(100),
  @Language varchar(20),
  @ProgramStart date,
  @ProgramEnd date,
  @ProgramPrice money,
  @LecturerID int,
  @TranslatorID int = NULL,
  @Syllabus varchar(255),
  @Place varchar(20),
  @MaxParticipants int,
  @EntryFee money


AS
BEGIN
  SET NOCOUNT ON;
  BEGIN TRY
      BEGIN TRANSACTION

      DECLARE @NewStudiesID int;
    
      IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @LecturerID)
      BEGIN
          THROW 50000, 'LecturerID does not exist in the Teachers table.', 1;
      END;

      IF @TranslatorID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Translators WHERE TranslatorID = @TranslatorID)
      BEGIN
          THROW 50000, 'TranslatorID does not exist in the Translators table.', 1;
      END;

      INSERT INTO Studies (Syllabus, Place, MaxParticipants, EntryFee)
      VALUES (@Syllabus, @Place, @MaxParticipants, @EntryFee);

      SELECT @NewStudiesID = ISNULL(MAX(StudiesID), 0)

      INSERT INTO EducationalPrograms (ProgramName, StudiesID, Language, ProgramStart, ProgramEnd, ProgramPrice, LecturerID, TranslatorID)
      VALUES (@ProgramName, @NewStudiesID, @Language, @ProgramStart, @ProgramEnd, @ProgramPrice, @LecturerID, @TranslatorID);

      COMMIT TRANSACTION
      PRINT 'Studies added successfully.';


  END TRY
  BEGIN CATCH
      ROLLBACK TRANSACTION
      DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
      THROW 52000, @msg, 1;
  END CATCH
END;
```

<div style="page-break-after: always;"></div>

## 15. Changing the details of an educational program
```sql
CREATE PROCEDURE UpdateEducationalProgram
   @ProgramID INT,
   @NewProgramName VARCHAR(100) = NULL,
   @NewLanguage VARCHAR(20) = NULL,
   @NewProgramStart DATE = NULL,
   @NewProgramEnd DATE = NULL,
   @NewProgramPrice MONEY = NULL,
   @NewLecturerID INT = NULL,
   @NewTranslatorID INT = NULL,
   @NewSyllabus VARCHAR(255) = NULL,
   @NewStudiesPlace VARCHAR(100) = NULL,
   @NewMinParticipants INT = NULL,
   @NewEntryFee MONEY = NULL,
   @NewCoursesPlace VARCHAR(40) = NULL,
   @NewAdvance MONEY = NULL,
   @NewClassID INT = NULL
AS
BEGIN
 BEGIN TRY
   BEGIN TRANSACTION

   SET NOCOUNT ON;
   DECLARE @IsCourse BIT, @IsWebinar BIT, @IsStudies BIT

   -- Sprawdzenie typu programu na podstawie ProgramID
   SELECT @IsCourse = IIF(EXISTS (SELECT 1 FROM EducationalPrograms WHERE ProgramID = @ProgramID and CourseID IS NOT NULL), 1, 0),
          @IsWebinar = IIF(EXISTS (SELECT 1 FROM EducationalPrograms WHERE ProgramID = @ProgramID and WebinarID IS NOT NULL), 1, 0),
          @IsStudies = IIF(EXISTS (SELECT 1 FROM EducationalPrograms WHERE ProgramID = @ProgramID and StudiesID IS NOT NULL), 1, 0)

   IF @IsStudies = 1 AND (@NewSyllabus IS NOT NULL OR @NewStudiesPlace IS NOT NULL OR @NewMinParticipants IS NOT NULL OR @NewEntryFee IS NOT NULL)
   BEGIN
       UPDATE Studies
       SET Syllabus = ISNULL(@NewSyllabus, Syllabus),
           Place = ISNULL(@NewStudiesPlace, Place),
           MaxParticipants = ISNULL(@NewMinParticipants, MaxParticipants),
           EntryFee = ISNULL(@NewEntryFee, EntryFee)
       FROM Studies
       JOIN EducationalPrograms ON Studies.StudiesID = EducationalPrograms.StudiesID
       WHERE EducationalPrograms.ProgramID = @ProgramID;
   END
   ELSE IF @IsStudies = 0 AND (@NewSyllabus IS NOT NULL OR @NewStudiesPlace IS NOT NULL OR @NewMinParticipants IS NOT NULL OR @NewEntryFee IS NOT NULL)
       THROW 52313, N'EducationalProgram is not ranked in Studies', 10;

   ELSE IF @IsCourse = 1 AND (@NewCoursesPlace IS NOT NULL OR @NewAdvance IS NOT NULL)
   BEGIN
       UPDATE Courses
       SET Place = ISNULL(@NewCoursesPlace, Place),
           Advance = ISNULL(@NewAdvance, Advance)
       FROM Courses
       JOIN EducationalPrograms ON Courses.CourseID = EducationalPrograms.CourseID
       WHERE EducationalPrograms.ProgramID = @ProgramID;
   END
   ELSE IF @IsCourse = 0 AND (@NewCoursesPlace IS NOT NULL OR @NewAdvance IS NOT NULL)
       THROW 52313, N'EducationalProgram is not ranked in Courses', 10;

   ELSE IF @IsWebinar = 1 AND @NewClassID IS NOT NULL
   BEGIN
       IF NOT EXISTS (SELECT 1 FROM Classes WHERE ClassID = @NewClassID)
           THROW 52314, N'NewClassID does not exist in Classes', 10;
       UPDATE Webinars
       SET ClassID = @NewClassID
       FROM Webinars
       JOIN EducationalPrograms ON Webinars.WebinarID = EducationalPrograms.WebinarID
       WHERE EducationalPrograms.ProgramID = @ProgramID;
   END
   ELSE IF @IsWebinar = 0 AND @NewClassID IS NOT NULL
       THROW 52313, N'EducationalProgram is not ranked in Webinars', 10;

   IF (@NewProgramName IS NOT NULL OR @NewLanguage IS NOT NULL OR @NewProgramStart IS NOT NULL OR
       @NewProgramEnd IS NOT NULL OR @NewProgramPrice IS NOT NULL OR @NewLecturerID IS NOT NULL OR
       @NewTranslatorID IS NOT NULL)
   BEGIN
       IF @NewLecturerID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @NewLecturerID)
           THROW 52314, N'NewLecturerID does not exist in Teachers', 10;
       IF @NewTranslatorID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Translators WHERE TranslatorID = @NewTranslatorID)
           THROW 52314, N'NewTranslatorID does not exist in Translators', 10;
       UPDATE EducationalPrograms
       SET ProgramName = ISNULL(@NewProgramName, ProgramName),
           Language = ISNULL(@NewLanguage, Language),
           ProgramStart = ISNULL(@NewProgramStart, ProgramStart),
           ProgramEnd = ISNULL(@NewProgramEnd, ProgramEnd),
           ProgramPrice = ISNULL(@NewProgramPrice, ProgramPrice),
           LecturerID = ISNULL(@NewLecturerID, LecturerID),
           TranslatorID = ISNULL(@NewTranslatorID, TranslatorID)
       WHERE ProgramID = @ProgramID
   END
   COMMIT TRANSACTION

 END TRY
 BEGIN CATCH
   ROLLBACK TRANSACTION
   DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
   THROW 123456, @Message, 10;
 END CATCH
END;
```

## 16. Adding payment to a submitted order
```sql
CREATE PROCEDURE AddPayment
   @OrderID INT,
   @SystemPaymentID VARCHAR(255),
   @PayFull Bit
AS
BEGIN
   BEGIN TRY
       IF NOT EXISTS (SELECT * FROM Orders WHERE OrderID = @OrderID)
       BEGIN
           THROW 51234, N'There is no order with such ID', 1;
       END
       BEGIN
           DECLARE @price INT;
           IF @PayFull = 1
           BEGIN
             SELECT @price = dbo.CalculateFullPriceForOrder(@OrderID)
           END

           ELSE
           BEGIN
             SELECT @price = dbo.CalculateEntryPriceForOrder(@OrderID)
           END

           INSERT INTO Payments (OrderId, Amount, Date, Status, SystemPaymentID )
           VALUES (@OrderID, @price, GETDATE(), 0, @SystemPaymentID)
       END
   END TRY
   BEGIN CATCH
       DECLARE @ErrorMessage NVARCHAR(1000) = N'Error: ' + ERROR_MESSAGE();
       THROW 52011, @ErrorMessage, 1;
   END CATCH
END;
```

<div style="page-break-after: always;"></div>

## 17. Adding new offline classes within a study program
```sql

CREATE PROCEDURE AddStudiesOfflineClasses
   @StudiesID int,
   @RoomNumber int,
   @MaxParticipants int,
   @TeacherID int,
   @SubjectID int,
   @StartTime datetime,
   @EndTime datetime,
   @ClassPrice money = NULL,
   @ModuleID int,
   @PractiseID int = NULL

AS
BEGIN
   SET NOCOUNT ON;
   BEGIN TRY
       BEGIN TRAN
       DECLARE @StudiesMaxParticipants int;
       DECLARE @NewClassID int;
       DECLARE @StudiesProgramStart date;
       DECLARE @StudiesProgramEnd date;

       SELECT @StudiesMaxParticipants = (
               SELECT MaxParticipants from Studies
               where StudiesID = @StudiesID)

       SELECT @StudiesProgramStart = (
               SELECT ProgramStart from EducationalPrograms
               where StudiesID = @StudiesID)

       SELECT @StudiesProgramEnd = (
               SELECT ProgramEnd from EducationalPrograms
               where StudiesID = @StudiesID)

       IF (@StudiesMaxParticipants > @MaxParticipants)
       BEGIN
           THROW 50000, 'Amount of each ClassesMaxParticipants should be equal or greater than StudiesMaxParticipants.', 1;
       END;

       IF NOT (@EndTime > @StartTime)
       BEGIN
           THROW 50000, 'EndTime of classes must be greater than StartTime', 1;
       END;


       IF NOT ((cast(@StartTime as date) BETWEEN @StudiesProgramStart AND @StudiesProgramEnd) AND (cast(@EndTime as date) BETWEEN @StudiesProgramStart AND @StudiesProgramEnd))
       BEGIN
           THROW 50000, 'Classes cannot be scheduled outside the duration of the EducationalProgram', 1;
       END;

       EXEC AddOfflineClass @RoomNumber, @MaxParticipants, @TeacherID, @SubjectID, @StartTime, @EndTime, @ClassPrice, @ModuleID, @PractiseID, @NewClassID OUTPUT

       COMMIT TRAN
       PRINT 'StudiesOfflineClasses added successfully.';

   END TRY
   BEGIN CATCH
       ROLLBACK TRANSACTION
       DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
       THROW 52000, @msg, 1;
   END CATCH
END;
```
## 18. Procedure to update a student's exam grade for specific studies. If the grade falls within the range of 50%-100%, the completion status of the corresponding program in RegisteredPrograms is updated via trigger #2
```sql
CREATE PROCEDURE SetExamMark(
   @StudentID int,
   @StudiesID int,
   @Mark int)
   AS
   BEGIN
      BEGIN TRY
          IF NOT @MARK BETWEEN 0 AND 100
          THROW 52313, N'Mark should be positive value between 0 and 100', 1;

          IF NOT EXISTS (SELECT 1 FROM Students WHERE StudentID = @StudentID)
          THROW 52313, N'StudentID does not exist in the Students table.', 1;

          IF NOT EXISTS (SELECT 1 FROM Studies WHERE StudiesID = @StudiesID)
          THROW 52313, N'StudiesID does not exist in the Studies table.', 1;

          IF NOT EXISTS (select ss.RegisteredProgramID
              from StudentStudies(@StudentID) as ss
               where ss.StudiesID = @StudiesID)
          THROW 52313, N'Student is not registered for this studies.', 1;

          IF EXISTS (select 1 FROM Exams where StudentID = @StudentID and StudiesID = @StudiesID)
          BEGIN
              UPDATE EXAMS
              SET Mark = @Mark
              WHERE StudentID = @StudentID and StudiesID = @StudiesID
              PRINT 'Exam updated successfully.';
          END

          ELSE
          BEGIN
              INSERT INTO Exams (StudiesID, StudentID, Mark) VALUES (@StudiesID, @StudentID, @Mark)
              PRINT 'Exam added successfully.';
          END
      END TRY

      BEGIN CATCH
           DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
           THROW 52000, @msg, 1;
      END CATCH
   END
```
<div style="page-break-after: always;"></div>

## 19. "Explicit" change of access status to an educational program, which can only be set explicitly by the school director
```sql
CREATE PROCEDURE SetProgramAccess(
  @RegisteredProgramID int,
  @Status int)
AS
BEGIN
   SET NOCOUNT ON;
   BEGIN TRY
       DECLARE @EducationalProgramID INT
       SELECT @EducationalProgramID = (select ProgramID from RegisteredPrograms where RegisteredProgramID = @RegisteredProgramID)
       IF NOT EXISTS (SELECT 1 FROM RegisteredPrograms WHERE RegisteredProgramID = @RegisteredProgramID)
       BEGIN
         THROW 50000, 'RegisteredProgramID does not exist in the RegisteredPrograms table.', 1;
       END;

       IF @Status != 0 AND @Status != 1
       BEGIN;
           throw 52000, N'Status should be equal to "1" to confirm access or "0" to cancel access', 1
       END

       IF @Status = 1 AND EXISTS (SELECT 1 FROM EducationalPrograms where ProgramID = @EducationalProgramID and StudiesID is not null)
       BEGIN
           DECLARE @MaxParticipants INT
           SELECT @MaxParticipants = (select MaxParticipants from EducationalPrograms inner join Studies on EducationalPrograms.StudiesID = Studies.StudiesID)

           IF (SELECT COUNT(*) FROM AllProgramParticipants(@EducationalProgramID) WHERE Access = 'true') = @MaxParticipants
           BEGIN
                THROW 52313, N'Participants number over MaxParticipants number for this Program', 1;
           END
       END

       BEGIN
           UPDATE RegisteredPrograms SET Access = @Status WHERE RegisteredProgramID = @RegisteredProgramID
       END
   END TRY
   BEGIN CATCH
     DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
     THROW 52000, @msg, 1;
   END CATCH
END
```
<div style="page-break-after: always;"></div>

## 20. "Explicit" change of access status to individual classes, which can only be set explicitly by the school director
```sql
CREATE PROCEDURE SetClassAccess(
  @RegisteredClassID int,
  @Status int)
AS
BEGIN
   SET NOCOUNT ON;
   BEGIN TRY
       DECLARE @ClassID INT
       SELECT @ClassID = (select ClassID from RegisteredClasses where RegisteredClassID = @RegisteredClassID)

       IF NOT EXISTS (SELECT 1 FROM RegisteredClasses WHERE RegisteredClassID = @RegisteredClassID)
       BEGIN
         THROW 50000, 'RegisteredClassID does not exist in the RegisteredClasses table.', 1;
       END;

       IF @Status != 0 AND @Status != 1
       BEGIN;
           throw 52000, N'Status should be equal to "1" to confirm access or "0" to cancel access', 1
       END

       -- To oznacza, że próbujemy ustawić dostęp dla zajęć stacjonarnych, które posiadają limit miejsc
       IF @Status = 1 AND EXISTS (SELECT 1 FROM OfflineClasses where ClassID = @ClassID)
       BEGIN
           DECLARE @MaxParticipants INT
           SELECT @MaxParticipants = (select MaxParticipants from OfflineClasses where ClassID = @ClassID)

           IF (SELECT COUNT(*) FROM AllClassParticipants(@ClassID) WHERE Access = 'true') = @MaxParticipants
           BEGIN
                THROW 52313, N'Participants number over MaxParticipants number for this Classes', 1;
           END
       END
       BEGIN
           UPDATE RegisteredClasses SET Access = @Status WHERE RegisteredClassID = @RegisteredClassID
       END
   END TRY
   BEGIN CATCH
     DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
     THROW 52000, @msg, 1;
   END CATCH
END
```