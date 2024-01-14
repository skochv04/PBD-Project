## **Procedures**

### **1. Dodanie nowego Studenta**
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
       IF EXISTS(SELECT * FROM Students WHERE Email = @email)
           THROW 52034, N'Email already in use', 1;
       ELSE
           DECLARE @NewID INT;
           SELECT @NewID = ISNULL(MAX(StudentID),0) +1
           FROM Students


           -- Insert the new student into the Students table
           INSERT INTO Students (StudentID, FirstName, LastName, CountryID, Email)
           VALUES (@NewID, @firstName, @lastName, @countryID, @email);


           SELECT 'Student added successfully.' AS Message;
   END TRY
   BEGIN CATCH
       DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
       THROW 52034, @Message, 1;
   END CATCH
END
```

### **2. Dodanie nowego kursu**
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
   SET NOCOUNT ON;


   DECLARE @NewCourseID int;
   DECLARE @NewProgramID int;


   IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @LecturerID)
   BEGIN
       THROW 50000, 'TeacherID does not exist in the Teachers table.', 1;
   END;


   IF NOT EXISTS (SELECT 1 FROM Translators WHERE TranslatorID = @TranslatorID)
   BEGIN
       THROW 50000, 'TranslatorID does not exist in the Translators table.', 1;
   END;


   SELECT @NewCourseID = ISNULL(MAX(CourseID), 0) + 1
   FROM Courses;


   INSERT INTO Courses (CourseID, Place, Advance)
   VALUES (@NewCourseID, @Place, @Advance);


   SELECT @NewProgramID = ISNULL(MAX(ProgramID), 0) + 1
   FROM EducationalPrograms;


   INSERT INTO EducationalPrograms (ProgramID, ProgramName, CourseID, Language, ProgramStart, ProgramEnd, ProgramPrice, LecturerID, TranslatorID)
   VALUES (@NewProgramID, @ProgramName, @NewCourseID, @Language, @ProgramStart, @ProgramEnd, @ProgramPrice, @LecturerID, @TranslatorID);
   SELECT 'Course added successfully.' AS Message;
END TRY
BEGIN CATCH
    DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
    THROW 52011, @Message, 1;
END CATCH
END;
```

### **3. Usuwanie studenta**
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


### **4. Aktualizacja danych studenta (tylko email albo country)**
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
   END
END
```

### **5. Dodanie nowego nauczyciela**
```sql
   CREATE PROCEDURE AddTeacher(
  @firstName VARCHAR(20),
  @lastName VARCHAR(20),
  @countryID INT
)
AS
BEGIN
  DECLARE @NewID INT;
  SELECT @NewID = ISNULL(MAX(TeacherID),0) +1
  FROM Teachers




  INSERT INTO Teachers (TeacherID, FirstName, LastName, CountryID)
  VALUES (@NewID, @firstName, @lastName, @countryID)
  SELECT 'Teacher added successfully.' AS Message;
END
```

### **6. Odrobienie nieobecności**
```sql
	
create procedure redoAttendance @classID int, @studentID int
as
begin
   set nocount on
   begin try
       if not exists
           (
           select ClassID, ParticipantID
               from Attendance
           WHERE ClassID = @classID and ParticipantID = @studentID
           )
       begin;
           throw 52000, N'The student was not registered for the class with the given ID', 1
       end
       if exists
           (
           select ClassID, ParticipantID
               from Attendance
           WHERE ClassID = @classID and ParticipantID = @studentID and Redone = 1
           )
       begin;
           throw 52000, N'Attendance had already been made up earlier', 1
       end
       update Attendance
       set Redone = 1
       where ClassID = @classID and ParticipantID = @studentID
       print 'Attendance was successfully set as redone!'
   end try
   begin catch
       declare @error varchar(1000)= 'Error when setting attendance as made up: ' + ERROR_MESSAGE();
       throw 77777, @error, 1
   end catch
end
```

### **7. Dodawania zamówienia**
```sql

CREATE PROCEDURE AddOrder(
   @Studentid INT
)
AS
BEGIN
  BEGIN TRY
      IF EXISTS(SELECT * FROM Students WHERE StudentID = @Studentid)
      BEGIN
          DECLARE @NewID INT;
          SELECT @NewID = ISNULL(MAX(OrderID),0) +1
          FROM Orders




          -- Insert the new student into the Students table
          INSERT INTO Orders (OrderID, StudentID, OrderDate)
          VALUES (@NewID, @Studentid, GETDATE());


          SELECT 'Order added successfully.' AS Message;
       END
       ELSE
           THROW 52011, N'There is no student with such id', 1;
  END TRY
  BEGIN CATCH
      DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
      THROW 52011, @Message, 1;
  END CATCH
END
```

### **8. Rejestrowanie pojedynczych zajęć do zamówienia**
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




      DECLARE @NewID INT;
      SELECT @NewID = ISNULL(MAX(RegisteredClassID),0) +1
      FROM RegisteredClasses




      INSERT INTO RegisteredClasses (RegisteredClassID, OrderID, ClassID)
      VALUES (@NewID, @OrderID, @ClassID);
      SELECT 'Class added successfully.' AS Message;
 END TRY
 BEGIN CATCH
     DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
     THROW 52011, @Message, 1;
 END CATCH
END

```

### **9. Rejestrowanie Programu do zamówienia**
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
       THROW 52313, N'There is no EducationlProram with such id', 1;




      DECLARE @NewID INT;
      SELECT @NewID = ISNULL(MAX(RegisteredProgramID),0) + 1
      FROM RegisteredPrograms




      INSERT INTO RegisteredPrograms (RegisteredProgramID, OrderID, ProgramID, Passed, CertificateLink)
      VALUES (@NewID, @OrderID, @ProgramID, @Passed, @CertificateLink);
      SELECT 'Program added successfully.' AS Message;
 END TRY
 BEGIN CATCH
     DECLARE @Message NVARCHAR(1000) = N'error: ' + ERROR_MESSAGE();
     THROW 52011, @Message, 1;
 END CATCH
END;
```

### **10. Dodanie nowego pojedynczego niestacjonarnego zajęcia**
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
 DECLARE @NewOnlineClassID int;
 BEGIN TRY
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
     SELECT @NewClassID = ISNULL(MAX(ClassID), 0) + 1
     FROM Classes;
     INSERT INTO Classes (ClassID, TeacherID, SubjectID, StartTime, EndTime, ClassPrice)
     VALUES (@NewClassID, @TeacherID, @SubjectID, @StartTime, @EndTime, @ClassPrice);
     SELECT @NewOnlineClassID = ISNULL(MAX(OnlineClassID), 0) + 1
     FROM OnlineClasses;
     INSERT INTO OnlineClasses (OnlineClassID, ClassID, Link, Synch)
     VALUES (@NewOnlineClassID, @NewClassID, @Link, @Synch)
     IF @ModuleID IS NOT NULL
     BEGIN
         UPDATE Classes SET ModuleID = @ModuleID WHERE ClassID = @NewClassID
     END;
     IF @PractiseID IS NOT NULL
     BEGIN
         UPDATE Classes SET PractiseID = @PractiseID WHERE ClassID = @NewClassID
     END;
     PRINT 'OnlineClass added successfully.';
 END TRY
 BEGIN CATCH
     DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
     THROW 52000, @msg, 1;
 END CATCH
END;

```

### **11. Dodanie nowego pojedynczego stacjonarnego zajęcia**
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


  DECLARE @NewOfflineClassID int;


  BEGIN TRY
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


      SELECT @NewClassID = ISNULL(MAX(ClassID), 0) + 1
      FROM Classes;


      INSERT INTO Classes (ClassID, TeacherID, SubjectID, StartTime, EndTime, ClassPrice, ModuleID)
      VALUES (@NewClassID, @TeacherID, @SubjectID, @StartTime, @EndTime, @ClassPrice, @ModuleID);


      SELECT @NewOfflineClassID = ISNULL(MAX(OfflineClassID), 0) + 1
      FROM OfflineClasses;


      INSERT INTO OfflineClasses (OfflineClassID, ClassID, RoomNumber, MaxParticipants)
      VALUES (@NewOfflineClassID, @NewClassID, @RoomNumber, @MaxParticipants)


      IF @PractiseID IS NOT NULL
      BEGIN
          UPDATE Classes SET PractiseID = @PractiseID WHERE ClassID = @NewClassID
      END;


      PRINT 'OfflineClass added successfully.';
  END TRY
  BEGIN CATCH
      DECLARE @msg NVARCHAR(2048) = N'ERROR: ' + ERROR_MESSAGE();
      THROW 52000, @msg, 1;
  END CATCH
END;
```

### **12. Dodanie nowego webinaru**
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


  DECLARE @NewClassID int;
  DECLARE @NewWebinarID int;
  DECLARE @NewProgramID int;


  IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @LecturerID)
  BEGIN
      THROW 50000, 'LecturerID does not exist in the Teachers table.', 1;
  END;


  IF @TranslatorID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Translators WHERE TranslatorID = @TranslatorID)
  BEGIN
      THROW 50000, 'TranslatorID does not exist in the Translators table.', 1;
  END;
  EXEC AddOnlineClass @Link, @Synch, @LecturerID, @SubjectID, @StartTime, @EndTime, @ClassPrice, @ModuleID, @PractiseID, @NewClassID OUTPUT


  SELECT @NewWebinarID = ISNULL(MAX(WebinarID), 0) + 1
  FROM Webinars;


  INSERT INTO Webinars (WebinarID, ClassID)
  VALUES (@NewWebinarID, @NewClassID);


  SELECT @NewProgramID = ISNULL(MAX(ProgramID), 0) + 1
  FROM EducationalPrograms;


  INSERT INTO EducationalPrograms (ProgramID, ProgramName, WebinarID, Language, ProgramStart, ProgramEnd, ProgramPrice, LecturerID, TranslatorID)
  VALUES (@NewProgramID, @ProgramName, @NewWebinarID, @Language, @ProgramStart, @ProgramEnd, @ProgramPrice, @LecturerID, @TranslatorID);




  PRINT 'Webinar added successfully.';
END;
```

### **13. Dodanie nowego kursu**
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


  DECLARE @NewCourseID int;
  DECLARE @NewProgramID int;


  IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @LecturerID)
  BEGIN
      THROW 50000, 'LecturerID does not exist in the Teachers table.', 1;
  END;


  IF @TranslatorID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Translators WHERE TranslatorID = @TranslatorID)
  BEGIN
      THROW 50000, 'TranslatorID does not exist in the Translators table.', 1;
  END;


  SELECT @NewCourseID = ISNULL(MAX(CourseID), 0) + 1
  FROM Courses;


  INSERT INTO Courses (CourseID, Place, Advance)
  VALUES (@NewCourseID, @Place, @Advance);


  SELECT @NewProgramID = ISNULL(MAX(ProgramID), 0) + 1
  FROM EducationalPrograms;


  INSERT INTO EducationalPrograms (ProgramID, ProgramName, CourseID, Language, ProgramStart, ProgramEnd, ProgramPrice, LecturerID, TranslatorID)
  VALUES (@NewProgramID, @ProgramName, @NewCourseID, @Language, @ProgramStart, @ProgramEnd, @ProgramPrice, @LecturerID, @TranslatorID);




  PRINT 'Course added successfully.';
END;
```

### **14. Dodanie nowych studiów**
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


  DECLARE @NewStudiesID int;
  DECLARE @NewProgramID int;


  IF NOT EXISTS (SELECT 1 FROM Teachers WHERE TeacherID = @LecturerID)
  BEGIN
      THROW 50000, 'LecturerID does not exist in the Teachers table.', 1;
  END;


  IF @TranslatorID IS NOT NULL AND NOT EXISTS (SELECT 1 FROM Translators WHERE TranslatorID = @TranslatorID)
  BEGIN
      THROW 50000, 'TranslatorID does not exist in the Translators table.', 1;
  END;


  SELECT @NewStudiesID = ISNULL(MAX(StudiesID), 0) + 1
  FROM Studies;


  INSERT INTO Studies (StudiesID, Syllabus, Place, MaxParticipants, EntryFee)
  VALUES (@NewStudiesID, @Syllabus, @Place, @MaxParticipants, @EntryFee);


  SELECT @NewProgramID = ISNULL(MAX(ProgramID), 0) + 1
  FROM EducationalPrograms;


  INSERT INTO EducationalPrograms (ProgramID, ProgramName, StudiesID, Language, ProgramStart, ProgramEnd, ProgramPrice, LecturerID, TranslatorID)
  VALUES (@NewProgramID, @ProgramName, @NewStudiesID, @Language, @ProgramStart, @ProgramEnd, @ProgramPrice, @LecturerID, @TranslatorID);




  PRINT 'Studies added successfully.';
END;
```

### **15. Zmiana szczegółów programu edukacyjnego (EducationalPrograms)**
```sql
```

### **16. Dodanie Płatności do złożonego zamówienia**
```sql

CREATE PROCEDURE AddPayment
   @OrderID INT,
   @SystemPaymentID VARCHAR(255),
   @PayFull Bit
AS
BEGIN
   BEGIN TRY
       -- Check if the OrderID exists in the Orders table
       IF EXISTS (SELECT * FROM Orders WHERE OrderID = @OrderID)
       BEGIN
           DECLARE @price INT;
           IF @PayFull = 1
           BEGIN
           SELECT @price = (
               SELECT SUM(ISNULL(EP.ProgramPrice,0)) + SUM(ISNULL(C.ClassPrice,0))
               FROM Orders O
               LEFT JOIN RegisteredPrograms RP ON RP.OrderID = O.OrderID
               LEFT JOIN RegisteredClasses RC ON O.OrderID = RC.OrderID
               LEFT JOIN Classes C ON RC.ClassID = C.ClassID
               LEFT JOIN EducationalPrograms EP ON EP.ProgramID = RP.ProgramID
               GROUP BY O.OrderID
               HAVING O.OrderId = @OrderID
           )
           END
           ELSE
           BEGIN
           SELECT @price = (
               SELECT SUM(ISNULL(S.EntryFee,0)) + SUM(ISNULL(CS.Advance,0)) + SUM(ISNULL(C.ClassPrice,0))
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
           )
           END
           DECLARE @NewID INT;
           SELECT @NewID = ISNULL(MAX(PaymentID),0) +1
           FROM Payments
           INSERT INTO Payments (PaymentID, OrderId, Amount, Date, Status, SystemPaymentID )
           VALUES (@NewID, @OrderID, @price, GETDATE(), 0, @SystemPaymentID)
       END
       ELSE
       BEGIN
           THROW 51234, N'There is no order with such ID', 1;
       END
   END TRY


   BEGIN CATCH
       DECLARE @ErrorMessage NVARCHAR(1000) = N'Error: ' + ERROR_MESSAGE();
       THROW 52011, @ErrorMessage, 1;
   END CATCH
END;
```

### **17. Anulowanie zamówienia**
```sql

CREATE PROCEDURE CancelOrder
   @OrderID int
AS
BEGIN
   -- Check if the order exists
   IF EXISTS (SELECT 1 FROM Orders WHERE OrderID = @OrderID)
   BEGIN
       -- Update the status and set OrderDate for the cancelled order
       UPDATE Orders
       SET OrderDate = '2099-12-31' -- You can set it to a specific date in the future or use a special code
       WHERE OrderID = @OrderID;


       -- Optionally, you may want to perform additional actions like updating related tables or logging the cancellation.
      
       -- Print a message indicating the order has been cancelled
       PRINT 'Order ' + CAST(@OrderID AS varchar(10)) + ' has been cancelled.';


   END
   ELSE
   BEGIN
       -- Print a message if the order does not exist
       THROW 52341, N'Order does not exits', 1;
   END
END;
```

### **18. Wyswietl listę obecności dla wydarzenia**
```sql

CREATE PROCEDURE GetAttendanceReport
   @ClassID int
AS
BEGIN
   IF EXISTS ( SELECT * FROM Classes WHERE ClassID = @ClassID)
   BEGIN
       SELECT
           s.FirstName + ' ' + s.LastName AS StudentName,
           a.Present,
           a.Redone
       FROM Attendance a
       JOIN Students s ON a.ParticipantID = s.StudentID
       WHERE a.ClassID = @ClassID;
   END
   ELSE
   BEGIN
       THROW 53523, N'There is no such class', 1;
   END
END;
```

### **19. Dodanie nowych offline zajęć w ramach studiów**
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


  DECLARE @StudiesMaxParticipants int;
  DECLARE @NewClassID int;


  SELECT @StudiesMaxParticipants = (
          SELECT MaxParticipants from Studies
          where StudiesID = @StudiesID)
  IF (@StudiesMaxParticipants > @MaxParticipants)
  BEGIN
      THROW 50000, 'Amount of each ClassesMaxParticipants should be equal or greater than StudiesMaxParticipants.', 1;
  END;


  EXEC AddOfflineClass @RoomNumber, @MaxParticipants, @TeacherID, @SubjectID, @StartTime, @EndTime, @ClassPrice, @ModuleID, @PractiseID, @NewClassID OUTPUT
  PRINT 'StudiesOfflineClasses added successfully.';
END;

```

### **20. ...**
```sql
```

### **21. ...**
```sql
```