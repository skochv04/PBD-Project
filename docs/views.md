# Views

## 1. Financial Reports – revenue summary for each webinar/course/study program
```sql
-- Webinars
CREATE VIEW WebinarsRevenue AS
SELECT Webinars.WebinarID, EducationalPrograms.ProgramName, EducationalPrograms.ProgramStart AS WebinarStart, EducationalPrograms.ProgramEnd AS WebinarEnd, COALESCE(SUM(Payments.Amount),0) AS Revenue
FROM Webinars
LEFT JOIN EducationalPrograms ON Webinars.WebinarID = EducationalPrograms.WebinarID
LEFT JOIN RegisteredPrograms ON RegisteredPrograms.ProgramID = EducationalPrograms.ProgramID
LEFT JOIN Orders ON Orders.OrderID = RegisteredPrograms.OrderID
LEFT JOIN Payments ON Orders.OrderID = Payments.OrderID
GROUP BY Webinars.WebinarID, EducationalPrograms.ProgramName, EducationalPrograms.ProgramStart, EducationalPrograms.ProgramEnd
```
![WebinarsRevenue](img/WebinarsRevenue.png)

```sql
--Courses
CREATE VIEW CoursesRevenue AS SELECT Courses.CourseID, EducationalPrograms.ProgramName, EducationalPrograms.ProgramStart as CourseStart, EducationalPrograms.ProgramEnd as CourseEnd, COALESCE(SUM(Payments.Amount),0) AS Revenue
FROM Courses
LEFT JOIN EducationalPrograms ON Courses.CourseID = EducationalPrograms.CourseID
LEFT JOIN RegisteredPrograms ON RegisteredPrograms.ProgramID = EducationalPrograms.ProgramID
LEFT JOIN Orders ON Orders.OrderID = RegisteredPrograms.OrderID
LEFT JOIN Payments ON Orders.OrderID = Payments.OrderID
GROUP BY Courses.CourseID, EducationalPrograms.ProgramName, EducationalPrograms.ProgramStart, EducationalPrograms.ProgramEnd;
```
![CoursesRevenue](img/CoursesRevenue.png)

<div style="page-break-after: always;"></div>

```sql
-- Studies
CREATE VIEW StudiesRevenue AS SELECT Studies.StudiesID, EducationalPrograms.ProgramName, EducationalPrograms.ProgramStart as StudiesStart, EducationalPrograms.ProgramEnd as StudiesEnd, COALESCE(SUM(Payments.Amount),0) AS Revenue
FROM Studies
LEFT JOIN EducationalPrograms ON Studies.StudiesID = EducationalPrograms.StudiesID
LEFT JOIN RegisteredPrograms ON RegisteredPrograms.ProgramID = EducationalPrograms.ProgramID
LEFT JOIN Modules ON Modules.ProgramID = EducationalPrograms.ProgramID
LEFT JOIN Practises ON Practises.StudiesID = Studies.StudiesID
LEFT JOIN Classes ON Classes.ModuleID = Modules.ModuleID OR Classes.PractiseID = Practises.PractiseID
LEFT JOIN RegisteredClasses ON RegisteredClasses.ClassID = Classes.ClassID
LEFT JOIN Orders ON Orders.OrderID = RegisteredClasses.OrderID OR Orders.OrderID = RegisteredPrograms.OrderID
LEFT JOIN Payments ON Payments.OrderID = Orders.OrderID
GROUP BY Studies.StudiesID, EducationalPrograms.ProgramName, EducationalPrograms.ProgramStart, EducationalPrograms.ProgramEnd;
```
![StudiesRevenue](img/StudiesRevenue.png)



## 2. "Debtors" list – individuals who have used the services but have not made the payments
```sql
CREATE VIEW Debtors AS
    SELECT S.*, ISNULL(OPS.ProgramsCost, 0) + ISNULL(OCS.ClassesCost, 0) - ISNULL(OP.Paid, 0) As Debt
    FROM Students S
    LEFT JOIN (
        SELECT O.StudentID, SUM(EP.ProgramPrice) AS ProgramsCost
        FROM Orders O
        LEFT JOIN RegisteredPrograms RP ON RP.OrderID = O.OrderID
        LEFT JOIN EducationalPrograms EP ON EP.ProgramID = RP.ProgramID
        GROUP BY O.StudentID
    ) OPS ON OPS.StudentID = S.StudentID
    LEFT JOIN (
        SELECT O.StudentID, SUM(C.ClassPrice) AS ClassesCost
        FROM Orders O
        LEFT JOIN RegisteredClasses RC ON RC.OrderID = O.OrderID
        LEFT JOIN Classes C ON C.ClassID = RC.ClassID
        GROUP BY O.StudentID
    ) OCS ON OCS.StudentID = S.StudentID
    LEFT JOIN (
        SELECT O.StudentID, SUM(P.Amount) AS Paid
        FROM Orders O
        LEFT JOIN Payments P ON P.OrderID = O.OrderID
        GROUP BY O.StudentID
    ) OP ON OP.StudentID = S.StudentID
    WHERE ISNULL(OPS.ProgramsCost, 0) + ISNULL(OCS.ClassesCost, 0) > ISNULL(OP.Paid, 0)
```
![Debtors](img/Debtors.png)

## 3. General report on the number of registered participants for upcoming events – future classes (with information on whether the classes are in-person or remote)
```sql
CREATE VIEW NumOfInterestedInFutureClasses as
with
tab as (
   select Classes.ClassID, count(Students.StudentID) as NumOfInterested
   from Classes
       left outer join RegisteredClasses RC on Classes.ClassID = RC.ClassID and RC.Access = 'true'
       left outer join Orders on RC.OrderID = Orders.OrderID
       left outer join Students on Orders.StudentID = Students.StudentID
   where Classes.StartTime > getdate()
   group by Classes.ClassID)
select Classes.ClassID, Classes.StartTime, Teachers.FirstName + ' ' + Teachers.LastName as Teacher, Subjects.SubjectName, tab.NumOfInterested, OfflineClassID, OnlineClassID
from Classes
   left outer join tab on tab.ClassID = Classes.ClassID
   left outer join OnlineClasses on Classes.ClassID = OnlineClasses.ClassID
   left outer join OfflineClasses on Classes.ClassID = OfflineClasses.ClassID
   left outer join Teachers on Classes.TeacherID = Teachers.TeacherID
   left outer join Subjects on Classes.SubjectID = Subjects.SubjectID
where Classes.StartTime > getdate()
```
![NumOfInterestedInFutureClasses](img/NumOfInterestedInFutureClasses.png)


## 4. General report on the number of registered participants for upcoming educational programs, along with their start dates
```sql
CREATE VIEW NumOfInterestedInFutureEducationalPrograms as
with tab as (
select EducationalPrograms.ProgramID, count(Students.StudentID) as NumOfInterested
from EducationalPrograms
   left outer join RegisteredPrograms RP on EducationalPrograms.ProgramID = RP.ProgramID and RP.Access = 'true'
   left outer join Orders on RP.OrderID = Orders.OrderID
   left outer join Students on Orders.StudentID = Students.StudentID
where EducationalPrograms.ProgramStart > getdate()
group by EducationalPrograms.ProgramID
) select tab.ProgramID, EducationalPrograms.ProgramName, tab.NumOfInterested, EducationalPrograms.ProgramStart
from EducationalPrograms
   join tab on EducationalPrograms.ProgramID = tab.ProgramID
```
![NumOfInterestedInFutureEducationalPrograms](img/NumOfInterestedInFutureEducationalPrograms.png)

<div style="page-break-after: always;"></div>

## 5. General report on the list of individuals registered for in-person classes within an educational program
```sql

CREATE VIEW OfflineParticipantsList as
select st.StudentID, st.FirstName + ' ' + st.LastName as Student, c.ClassID, m.ModuleName, sub.SubjectName, t.FirstName + ' ' + t.LastName as Teacher, c.StartTime, c.EndTime, ofl.RoomNumber, rp.Access
from Students as st
    inner join Orders as od
        on st.StudentID = od.StudentID
    inner join RegisteredPrograms as rp
        on od.OrderID = rp.OrderID
    inner join EducationalPrograms as ep
        on rp.ProgramID = ep.ProgramID
    inner join Modules as m
        on ep.ProgramID = m.ProgramID
    inner join Classes as c
        on m.ModuleID = c.ModuleID
    inner join Subjects as sub
        on c.SubjectID = sub.SubjectID
    inner join Teachers as t
        on c.TeacherID = t.TeacherID
    inner join OfflineClasses as ofl
        on c.ClassID = ofl.ClassID
```
![OfflineParticipantsList](img/OfflineParticipantsList.png)

## 6. Attendance list for each training session, including date, first name, last name, and information on whether the participant was present or absent
```sql

CREATE VIEW AttendanceAllClasses as 
SELECT C.ClassID, CONCAT(year(C.StartTime), '-', month(C.StartTime), '-', day(C.StartTime)) as Date, Students.FirstName + ' ' + Students.LastName as Student, A.Present
FROM Classes C
LEFT OUTER JOIN Attendance A on C.ClassID = A.ClassID
LEFT OUTER JOIN Students on Students.StudentID = A.ParticipantID
```
![AttendanceAllClasses](img/AttendanceAllClasses.png)

<div style="page-break-after: always;"></div>

## 7. Bilocation report: List of conflicting classes along with student information, class ID, and conflicting schedules
```sql
CREATE VIEW BilocationsList as select distinct s.StudentID, s.FirstName + ' ' + s.LastName as Student, a.ClassID as a_ClassID, a.StartTime as a_StartTime, a.EndTime as a_EndTime, b.ClassID as b_ClassID, b.StartTime as b_StartTime, b.EndTime as b_EndTime from Students as s
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
```
![BilocationsList](img/BilocationsList.png)

## 8. Number of participants for each completed event
```sql
CREATE VIEW NumberOfParticipations as select c.ClassID, c.TeacherID, t.FirstName + ' ' + t.LastName as Teacher, sub.SubjectName, c.StartTime, c.EndTime, count(s.StudentID) as StudentsAmount
   from Classes as c
       left join Attendance as a
           on c.ClassID = a.ClassID
       left join Students as s
           on a.ParticipantID = s.StudentID
       left join Teachers as t
           on c.TeacherID = t.TeacherID
       left join Subjects as sub
           on c.SubjectID = sub.SubjectID
   where c.EndTime < getdate()
group by c.ClassID, c.TeacherID, t.FirstName + ' ' + t.LastName, sub.SubjectName, c.SubjectID, c.StartTime, c.EndTime
```
![NumberOfParticipations](img/NumberOfParticipations.png)

<div style="page-break-after: always;"></div>

## 9. Data for each conducted exam, including the grade, student ID, study program ID, educational program ID, and the start & end dates of the respective studies
```sql
CREATE VIEW ExamDetails as select ex.ExamID, ex.StudentID, S2.FirstName + ' ' + S2.LastName as Student, ex.Mark, ex.StudiesID, ep.ProgramName, ep.ProgramStart, ep.ProgramEnd from exams as ex
   inner join Studies as s
       on ex.StudiesID = s.StudiesID
   inner join dbo.Students S2
       on ex.StudentID = S2.StudentID
   inner join dbo.EducationalPrograms EP
       on s.StudiesID = EP.StudiesID
```
![ExamDetails](img/ExamDetails.png)

## 10. List of subjects conducted within the modules of specific study programs, along with information about the category and the teacher for each subject
```sql
create view StudiesSubjectsInfo as select ed.ProgramID, ed.ProgramName, m.ModuleName, sub.SubjectName, sc.CategoryName, t.FirstName + ' ' + t.LastName as Teacher
   from Studies as s
       inner join EducationalPrograms as ed
           on s.StudiesID = ed.StudiesID
       inner join Modules as m
           on ed.ProgramID = m.ProgramID
       inner join Classes as c
           on m.ModuleID = c.ModuleID
       inner join Subjects as sub
           on c.SubjectID = sub.SubjectID
       inner join SubjectCategories as sc
           on sub.CategoryID = sc.CategoryID
       inner join Teachers as t
           on c.TeacherID = t.TeacherID
```
![StudiesSubjectsInfo](img/StudiesSubjectsInfo.png)

<div style="page-break-after: always;"></div>

## 11. List of subjects conducted within the modules of specific courses, along with information about the category and the teacher for each subject
```sql
create view CoursesSubjectsInfo as select ed.ProgramID, ed.ProgramName, m.ModuleName, sub.SubjectName, sc.CategoryName, t.FirstName + ' ' + t.LastName as Teacher
  from Courses
      inner join EducationalPrograms as ed
          on Courses.CourseID = ed.CourseID
      inner join Modules as m
          on ed.ProgramID = m.ProgramID
      inner join Classes as c
          on m.ModuleID = c.ModuleID
      inner join Subjects as sub
          on c.SubjectID = sub.SubjectID
      inner join SubjectCategories as sc
          on sub.CategoryID = sc.CategoryID
      inner join Teachers as t
          on c.TeacherID = t.TeacherID

```
![CoursesSubjectsInfo](img/CoursesSubjectsInfo.png)

## 12. List of upcoming webinars, displaying the subject name, the first and last name of the teacher conducting the webinar, the duration of the webinar, and the access period to the webinar
```sql
create view WebinarsInfo as select w.WebinarID, c.ClassID, s.SubjectName, t.FirstName + ' ' + t.LastName as TeacherName, c.StartTime as WebinarStart, c.EndTime as WebinarEnd, ed.ProgramStart as AccessStart, ed.ProgramEnd as AccessEnd
   from webinars as w
       inner join dbo.Classes C on C.ClassID = w.ClassID
       inner join dbo.OnlineClasses OC on C.ClassID = OC.ClassID
       inner join dbo.Teachers T on C.TeacherID = T.TeacherID
       inner join EducationalPrograms as Ed ON w.WebinarID = Ed.WebinarID
       inner join Subjects as s on C.SubjectID = s.SubjectID
```
![WebinarsInfo](img/WebinarsInfo.png)

<div style="page-break-after: always;"></div>

## 13. List of educational programs that students are enrolled in, along with the order number in which the program was purchased, the start and end dates of the program, and completion status
```sql
create view StudentsPrograms as select s.StudentID, s.FirstName + ' ' + s.LastName as Student, rp.RegisteredProgramID, ep.ProgramID, ep.ProgramName, ep.ProgramStart, ep.ProgramEnd, rp.Access, rp.Passed
    from Students as s
        inner join Orders as o
            on s.StudentID = o.StudentID
        inner join RegisteredPrograms as rp
            on o.OrderID = rp.OrderID
        inner join EducationalPrograms as ep
            on rp.ProgramID = ep.ProgramID
```
![StudentsPrograms](img/StudentsPrograms.png)

## 14. List of individuals enrolled in the specific session "externally". The view displays the student's first and last name, the order number in which the session was purchased, the session number, access status, the subject name for the session, the module name under which the session is conducted, and the name of the practice (if the session is a practice)
```sql
create view StudentsOuterClasses as select s.StudentID, s.FirstName + ' ' + s.LastName as Student, o.OrderID, c.ClassID, m.ModuleName, sub.SubjectName, t.FirstName + ' ' + t.LastName as Teacher, c.StartTime, c.EndTime, ofl.RoomNumber, rc.Access
   from Students as s
       inner join Orders as o
           on s.StudentID = o.StudentID
       inner join RegisteredClasses as rc
           on o.OrderID = rc.OrderID
       inner join Classes as c
           on rc.ClassID = c.ClassID
       inner join Modules as m
           on c.ModuleID = m.ModuleID
       left join Practises as p
           on c.PractiseID = p.PractiseID
       inner join Subjects as sub
           on c.SubjectID = sub.SubjectID
       inner join Teachers as t
           on c.TeacherID = t.TeacherID
       inner join OfflineClasses as ofl
           on c.ClassID = ofl.ClassID
```
![StudentsOuterClasses](img/StudentsOuterClasses.png)