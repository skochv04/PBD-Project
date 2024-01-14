## **Funkcje**

### **1.Obliczanie średniej ocen dla studenta**
```sql

CREATE FUNCTION CalculateAverageGradeForStudent
(
   @StudentID int
)
RETURNS DECIMAL(5, 2) -- Zakładam, że oceny są przechowywane jako liczby zmiennoprzecinkowe
AS
BEGIN
   DECLARE @AverageGrade DECIMAL(5, 2);


   SELECT @AverageGrade = AVG(Mark)
   FROM Exams
   WHERE StudentID = @StudentID;


   RETURN ISNULL(@AverageGrade, 0); -- Jeśli nie ma ocen, zwraca 0
END;
```

### **2. Liczba studentów obecnych na zajęciach**
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

### **3. Obliczanie ilości dni pozostałych do zakończenia programu edukacyjnego**
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


   -- Jeżeli program już się zakończył, zwróć 0
   IF @DaysRemaining < 0
       SET @DaysRemaining = 0;


   RETURN @DaysRemaining;
END;
```

### **4.Obliczanie łącznej kwoty wydanej przez danego studenta na Programy edukacyjne**
```sql

CREATE FUNCTION CalculateTotalPaymentsForStudent
(
   @StudentID int
)
RETURNS MONEY
AS
BEGIN
   DECLARE @TotalPayments MONEY;


   SELECT @TotalPayments = SUM(Amount)
   FROM Payments
   WHERE OrderID IN (SELECT OrderID FROM Orders WHERE StudentID = @StudentID) AND status = 1;


   RETURN ISNULL(@TotalPayments, 0);
END;
```

### **5. ...**
### **6. ...**
### **7. ...**
### **8. ...**
### **9. ...**
### **10. ...**
### **11. ...**
### **12. ...**
### **13. ...**
### **14. ...**
### **15. ...**
### **16. ...**
### **17. ...**
### **18. ...**