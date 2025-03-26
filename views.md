## **Widoki**

### **1. Raporty finansowe – zestawienie przychodów dla każdego webinaru/kursu/studium**
```sql
-- Webinars
SELECT Webinars.WebinarID, EducationalPrograms.ProgramName, SUM(Payments.Amount) AS Revenue
FROM Webinars
JOIN EducationalPrograms ON Webinars.WebinarID = EducationalPrograms.WebinarID
JOIN RegisteredPrograms ON RegisteredPrograms.ProgramID = EducationalPrograms.ProgramID
JOIN Orders ON Orders.OrderID = RegisteredPrograms.OrderID
JOIN Payments ON Orders.OrderID = Payments.OrderID
GROUP BY Webinars.WebinarID, EducationalPrograms.ProgramName;

--Courses
SELECT Courses.CourseID, EducationalPrograms.ProgramName, SUM(Payments.Amount) AS Revenue
FROM Courses
INNER JOIN EducationalPrograms ON Courses.CourseID = EducationalPrograms.CourseID
JOIN RegisteredPrograms ON RegisteredPrograms.ProgramID = EducationalPrograms.ProgramID
JOIN Orders ON Orders.OrderID = RegisteredPrograms.OrderID
JOIN Payments ON Orders.OrderID = Payments.OrderID
GROUP BY Courses.CourseID, EducationalPrograms.ProgramName;

-- Studies
SELECT Studies.StudiesID, EducationalPrograms.ProgramName, SUM(Payments.Amount) AS Revenue
FROM Studies
JOIN EducationalPrograms ON Studies.StudiesID = EducationalPrograms.StudiesID
JOIN RegisteredPrograms ON RegisteredPrograms.ProgramID = EducationalPrograms.ProgramID
JOIN Modules ON Modules.ProgramID = EducationalPrograms.ProgramID
JOIN Practises ON Practises.StudiesID = Studies.StudiesID
JOIN Classes ON Classes.ModuleID = Modules.ModuleID OR Classes.PractiseID = Practises.PractiseID
JOIN RegisteredClasses ON RegisteredClasses.ClassID = Classes.ClassID
JOIN Orders ON Orders.OrderID = RegisteredClasses.OrderID OR Orders.OrderID = RegisteredPrograms.OrderID
JOIN Payments ON Payments.OrderID = Orders.OrderID
GROUP BY Studies.StudiesID, EducationalPrograms.ProgramName;
```

### **2. Lista „dłużników” – osoby, które skorzystały z usług, ale nie uiściły opłat.**
```sql
RAPORTOWANIE 2
;WITH OrderProgramSum AS(
SELECT O.StudentID, SUM(EP.ProgramPrice) AS ProgramsCost
FROM Orders O
JOIN RegisteredPrograms RP ON RP.OrderID = O.OrderID
JOIN EducationalPrograms EP ON EP.ProgramID = RP.ProgramID
GROUP BY O.StudentID
),
OrderClassesSum AS(
SELECT O.StudentID, SUM(C.ClassPrice) AS ClassesCost
FROM Orders O
JOIN RegisteredClasses RC ON RC.OrderID = O.OrderID
JOIN Classes  C ON C.ClassID = RC.ClassID
GROUP BY O.StudentID
),
OrderPaid AS(
SELECT O.StudentID, Sum(P.Amount) AS Paid
FROM Orders O
JOIN Payments P ON P.OrderID = O.OrderID
GROUP BY O.StudentID
)

```

### **3. Ogólny raport dotyczący frekwencji na zakończonych już wydarzeniach..**
```sql

-- a) Lista osób
select a.ClassID, s.StudentID, s.FirstName + ' ' + s.LastName as Student, c.TeacherID, c.SubjectID, c.StartTime, c.EndTime
    from Attendance as a
        inner join Students as s
            on a.ParticipantID = s.StudentID
        inner join Classes as c
            on a.ClassID = c.ClassID
    where c.EndTime < getdate()


-- b) Liczba osób dla każdego wydarzenia
select a.ClassID, c.TeacherID, c.SubjectID, c.StartTime, c.EndTime, count(s.StudentID) as StudentsAmount
    from Classes as c
        left join Attendance as a
            on c.ClassID = a.ClassID
        inner join Students as s
            on a.ParticipantID = s.StudentID
    where c.EndTime < getdate()
group by a.ClassID, c.TeacherID, c.SubjectID, c.StartTime, c.EndTime
```

### **4. Raport bilokacji: lista osób, które są zapisane na co najmniej dwa przyszłe szkolenia, które ze sobą kolidują czasowo.**
```sql
select * from (select s.StudentID, s.FirstName + ' ' + s.LastName as Student, count(a.ClassID) as Bilocations
from Students as s
       inner join Orders as o
           on s.StudentID = o.StudentID
       inner join RegisteredPrograms as rp
           on o.OrderID = rp.OrderID
       inner join Modules as m
           on rp.ProgramID = m.ProgramID
       inner join Classes as a
           on m.ModuleID = a.ModuleID
       cross join Classes as b
where a.ClassID < b.ClassID and ((a.StartTime BETWEEN b.StartTime and b.EndTime) or (b.StartTime BETWEEN a.StartTime and a.EndTime) or (a.EndTime BETWEEN b.StartTime and b.EndTime) or (b.EndTime BETWEEN a.StartTime and a.EndTime))
group by s.StudentID, s.FirstName + ' ' + s.LastName) as start
where Bilocations >= 2
```
