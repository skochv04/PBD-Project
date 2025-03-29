# 🎓 Learning Platform DB

A database model for a hybrid learning platform, developed during the Database Systems course in the 3rd year of university studies. Designed in **MS SQL Server**, including tables, relationships, procedures, functions, triggers, indexes and roles.

## 👥 Team Members:
- **Stas Kochevenko** | skochv04@gmail.com
- **Olgierd Smyka**
- **Bernard Gawor**

## 1. **Platform functionality overview**

### **Users**

**Unregistered client:**

- Viewing available services
- Creating an individual or business account

**Student (client with an account):**

- Viewing available services
- Ability to enroll in free or paid educational programs and classes in selected studies by making a payment
- Viewing all educational programs the student is enrolled in, along with their details
- Access to learning materials in courses and studies
- Viewing the schedule of classes the student is enrolled in

**Educator (teacher/translator):**

- Viewing all educational programs they lead, along with their details
- Access to learning materials for educational programs and the ability to edit them
- Ability to approve attendance lists for classes
- Ability to assign grades for classes and exams for students in the study program (teacher only)
- Ability to approve make-up classes for student absences

**System Administrator:**

- Managing the class schedule for educational programs
- Updating available services
- Ability to generate financial reports
- Access to statistical data
- Ability to modify student data
- Editing the employee list

**School Director:**

- Has the privileges of a system administrator
- Changing user access to educational programs


### **System**

- Financial reports – summary of revenues for each webinar/course/study program
- List of "debtors" – people who have used services but have not made payments
- General report on the number of registered participants for upcoming events (with information on whether the event is in-person or remote)
- General report on attendance for completed events
- Attendance list for each training session, including date, first name, last name, and attendance status (present or absent)
- Location conflict report: list of people enrolled in at least two upcoming training sessions that overlap in time
- Overview of externally registered students
- Overview of details for specific educational programs
- Automatic update of access to purchased educational programs and classes after a successful transaction
- Verification of the possibility to change access to an educational program or one of classes
- Automatic status change for completing an educational program after receiving a passing grade on the exam


### **Specifications**

- Courses and study programs can be held: online, in-person, or hybrid
- In-person classes for courses and study programs have a participant limit
- Webinars are available to participants for 30 days
- Completing a course requires passing at least 80% of the modules
- For study programs, it is required to complete practical classes and maintain a minimum attendance of 80%, with absences being made up by attending classes or a commercial course on a similar topic
- Practical classes last for 14 days – 100% attendance is required
- It is possible to enroll in chosen events without attending the entire study program, with a different pricing structure for such enrollments
- The administrator can enroll clients in unpaid training courses in exceptional situations
- Participation in a course requires a deposit at the time of registration, and the full payment must be made no later than 3 days before the course begins
- Participation in the study program requires an enrollment fee and payment for the specific session no later than 3 days before it starts
- Trainings may be conducted in various pre-determined languages
- All online classes are held on an external cloud platform
- The payment system is provided by an external company

<div style="page-break-after: always;"></div>

## **2. Database Schema**
![dbschema](../img/database-schema.png)

The services offered by the company (various courses and trainings) are connected by **EducationalPrograms**. Each record represents either a study program (**Studies**), a course (**Courses**), or a webinar (**Webinars**). A list of all classes (sessions) is stored in the **Classes** table. These sessions can be in-person (**OfflineClasses**) or online (**OnlineClasses**).

**Courses** consist of **Modules**. Classes of these modules can be conducted in-person or online.

Similar to courses, study programs consist of **Modules** and also include **Practises**.

Students can make **Orders** and view the list of programs (**RegisteredPrograms**) and classes (**RegisteredClasses**) they are enrolled in.

<div style="page-break-after: always;"></div>

## 3. **Tabels**
```sql
-- Table:  Translators
CREATE TABLE  Translators (
   TranslatorID int AUTO_INCREMENT NOT NULL,
   FirstName varchar(20)  NOT NULL,
   LastName varchar(20)  NOT NULL,
   CountryID int  NOT NULL,
   CONSTRAINT Translator_pk PRIMARY KEY  (TranslatorID)
);

-- Table: Attendance
-- Contains information regarding the attendance of specific students from the Students table in the classes from the Classes table
CREATE TABLE Attendance (
   AttendanceID int AUTO_INCREMENT NOT NULL,
   ClassID int  NOT NULL,
   Present bit  NOT NULL DEFAULT 0,
   ParticipantID int  NOT NULL,
   Redone bit  NOT NULL DEFAULT 0,
   CONSTRAINT Attendance_pk PRIMARY KEY  (AttendanceID)
);

-- Table: Classes
-- A single session within an educational program (or a specific module in the case of courses or study programs) can be either online or offline
CREATE TABLE Classes (
   ClassID int AUTO_INCREMENT NOT NULL,
   TeacherID int  NOT NULL,
   SubjectID int  NOT NULL,
   StartTime datetime  NOT NULL,
   EndTime datetime  NOT NULL,
   ClassPrice money  NULL,
   ModuleID int  NULL,
   PractiseID int  NULL,
   CHECK (EndTime > StartTime),
   CHECK (ClassPrice >= 0),
   CONSTRAINT Classes_pk PRIMARY KEY  (ClassID)
);


-- Table: Countries
CREATE TABLE Countries (
   CountryID int AUTO_INCREMENT NOT NULL,
   CountryName int  NOT NULL UNIQUE,
   CONSTRAINT Countries_pk PRIMARY KEY  (CountryID)
);

-- Table: EducationalPrograms
-- Contains details of a specific educational program, which can be a study program from the Studies table, a course from the Courses table, or a webinar from the Webinars table. In each record, only one of the three values: StudiesID, WebinarID, or CourseID is not NULL
CREATE TABLE EducationalPrograms (
   ProgramID int AUTO_INCREMENT NOT NULL,
   ProgramName varchar(100)  NOT NULL UNIQUE,
   StudiesID int  NULL,
   WebinarID int  NULL,
   CourseID int  NULL,
   Language varchar(20)  NOT NULL,
   ProgramStart date  NOT NULL,
   ProgramEnd date  NOT NULL,
   ProgramPrice money  NOT NULL,
   LecturerID int  NOT NULL DEFAULT ‘Polish’,
   TranslatorID int  NULL,
   CHECK (ProgramEnd > ProgramStart),
   CHECK (ProgramPrice >= 0),
   CONSTRAINT EducationalPrograms_pk PRIMARY KEY  (ProgramID)
);

```
<div style="page-break-after: always;"></div>

```sql
-- Table: Courses
CREATE TABLE Courses (
   CourseID int AUTO_INCREMENT NOT NULL,
   Place varchar(40)  NOT NULL,
   Advance money  NOT NULL,
   CHECK (Advance >= 0),
   CONSTRAINT Courses_pk PRIMARY KEY  (CourseID)
);

-- Table: Exams
-- Contains exam results for students (from the Students table) enrolled in study programs (from the Studies table)
CREATE TABLE Exams (
   ExamID int AUTO_INCREMENT NOT NULL,
   StudiesID int  NOT NULL,
   StudentID int  NOT NULL,
   Mark int  NOT NULL DEFAULT 0,
   CHECK(Mark >= 0 AND Mark <= 100),
   CONSTRAINT Exams_pk PRIMARY KEY  (ExamID)
);

-- Table: Modules
-- A set of classes on a specific topic, not identical to the concept of a subject (one module can include classes from different subjects). It allows for the combination of classes with different forms of education (in-person, asynchronous online, synchronous online, hybrid).
-- For example:
-- The module "Programming in Mathematics" could include a series of classes from mathematical subjects, where problems are solved using written code
CREATE TABLE Modules (
   ModuleID int AUTO_INCREMENT NOT NULL,
   ProgramID int  NOT NULL,
   ModuleName varchar(40)  NOT NULL,
   ModuleDescription varchar(100)  NOT NULL,
   CONSTRAINT Modules_pk PRIMARY KEY  (ModuleID)
);


-- Table: OfflineClasses
-- A subset of Classes: classes conducted in offline (in-person) mode are always associated with a specific module
CREATE TABLE OfflineClasses (
   OfflineClassID int AUTO_INCREMENT NOT NULL,
   ClassID int  NOT NULL,
   RoomNumber int  NOT NULL,
   MaxParticipants int  NOT NULL DEFAULT 0,
   Mark int  NULL,
   CHECK(Mark >= 0 AND Mark <= 100),
   CONSTRAINT OfflineClasses_pk PRIMARY KEY  (OfflineClassID)
);


-- Table: OnlineClasses
-- A subset of Classes: classes conducted online. These include synchronous and asynchronous modules
CREATE TABLE OnlineClasses (
   OnlineClassID int AUTO_INCREMENT NOT NULL,
   ClassID int  NOT NULL,
   Link varchar(255)  NOT NULL,
   Synch bit  NOT NULL DEFAULT 0,
   CONSTRAINT OnlineClasses_pk PRIMARY KEY  (OnlineClassID)
);

```

<div style="page-break-after: always;"></div>

```sql
-- Table: Orders
-- A list of orders placed by students. Information about purchased programs and individual classes is stored in the RegisteredPrograms and RegisteredClasses tables, respectively
CREATE TABLE Orders (
   OrderID int AUTO_INCREMENT NOT NULL,
   StudentID int  NOT NULL,
   OrderDate datetime  NOT NULL DEFAULT GETDATE(),
   OrderStatus varchar(40) NOT NULL DEFAULT 'NOT PAID',
   CHECK(OrderStatus IN ('NOT PAID', 'ENTRY PAID', 'FULL PAID'))
   CONSTRAINT Orders_pk PRIMARY KEY  (OrderID)
);

-- Table: Payments
-- A list of payments made to partially or fully pay for an order from the Orders table. The Status column indicates whether the payment was successfully completed, while the SystemPaymentID column contains a link to the external payment system
CREATE TABLE Payments (
   PaymentID int AUTO_INCREMENT NOT NULL,
   OrderID int  NOT NULL,
   Amount money  NOT NULL,
   Date date  NOT NULL DEFAULT GETDATE(),
   Status bit  NOT NULL DEFAULT 0,
   CHECK (Amount >= 0),
   CONSTRAINT Payments_pk PRIMARY KEY  (PaymentID)
);

-- Table: Practises
-- Each study program may include multiple practical classes. The table stores the description and identifier of each practical classes. In the Classes table, there is a field PracticeID, which is not NULL when the specific classes are part of a given practical classes.
CREATE TABLE Practises (
   PractiseID int AUTO_INCREMENT NOT NULL,
   StudiesID int  NOT NULL,
   PracticeName varchar(40)  NOT NULL,
   PracticeDescription varchar(255)  NOT NULL,
   CONSTRAINT Practices_pk PRIMARY KEY  (PractiseID)
);

-- Table: RegisteredClasses
-- A list of individual classes (sessions within study programs) purchased by students, along with their order numbers
CREATE TABLE RegisteredClasses (
   RegisteredClassID int AUTO_INCREMENT NOT NULL,
   OrderID int  NOT NULL,
   ClassID int  NOT NULL,
   Access bit NOT NULL DEFAULT 0,
   CONSTRAINT RegisteredClasses_pk PRIMARY KEY  (RegisteredClassID)
);

-- Table: RegisteredPrograms
-- A list of EducationalPrograms purchased by students, along with their order numbers
CREATE TABLE RegisteredPrograms (
   RegisteredProgramID int AUTO_INCREMENT NOT NULL,
   OrderID int  NOT NULL,
   ProgramID int  NOT NULL,
   Passed bit  NOT NULL DEFAULT 0,
   CertificateLink varchar(255),
   Access bit NOT NULL DEFAULT 0,
   CONSTRAINT RegisteredPrograms_pk PRIMARY KEY  (RegisteredProgramID)
);

-- Table: SubjectCategories
-- Contains categories of various subjects conducted, as stored in the Subjects table. For example, Mathematics (SubjectCategories) is a category of the subject Algebra (Subjects)
CREATE TABLE SubjectCategories (
   CategoryID int AUTO_INCREMENT NOT NULL,
   CategoryName varchar(40)  NOT NULL UNIQUE,
   Description varchar(255)  NOT NULL,
   CONSTRAINT SubjectCategories_pk PRIMARY KEY  (CategoryID)
);
```

<div style="page-break-after: always;"></div>

```sql
-- Table: Studies
CREATE TABLE Studies (
   StudiesID int AUTO_INCREMENT NOT NULL,
   Syllabus varchar(255)  NOT NULL,
   Place varchar(100)  NOT NULL,
   MaxParticipants int  NOT NULL,
   EntryFee money  NOT NULL
   CHECK (EntryFee >= 0),
   CONSTRAINT Studies_pk PRIMARY KEY  (StudiesID)
);

-- Table: Students
CREATE TABLE Students (
   StudentID int AUTO_INCREMENT NOT NULL,
   FirstName varchar(20)  NOT NULL,
   LastName varchar(20)  NOT NULL,
   CountryID int  NOT NULL,
   Email varchar(40)  NOT NULL UNIQUE,
   CONSTRAINT Students_pk PRIMARY KEY  (StudentID)
);

-- Table: Subjects
CREATE TABLE Subjects (
   SubjectID int AUTO_INCREMENT NOT NULL,
   CategoryID int  NOT NULL,
   Description varchar(255)  NOT NULL,
   SubjectName varchar(40)  NOT NULL UNIQUE,
   CONSTRAINT Subjects_pk PRIMARY KEY  (SubjectID)
);

-- Table: Teachers
CREATE TABLE Teachers (
   TeacherID int AUTO_INCREMENT NOT NULL,
   FirstName varchar(15)  NOT NULL,
   LastName varchar(15)  NOT NULL,
   CountryID int  NOT NULL,
   CONSTRAINT Teachers_pk PRIMARY KEY  (TeacherID)
);

-- Table: Webinars
CREATE TABLE Webinars (
   WebinarID int AUTO_INCREMENT NOT NULL,
   ClassID int  NOT NULL,
   CONSTRAINT Webinars_pk PRIMARY KEY  (WebinarID)
);

-- foreign keys
-- Reference:  Translators_Countries (table:  Translators)
ALTER TABLE  Translators ADD CONSTRAINT  Translators_Countries
   FOREIGN KEY (CountryID)
   REFERENCES Countries (CountryID);


-- Reference: Attendance_Students (table: Attendance)
ALTER TABLE Attendance ADD CONSTRAINT Attendance_Students
   FOREIGN KEY (ParticipantID)
   REFERENCES Students (StudentID);


-- Reference: Classes_Attendance (table: Attendance)
ALTER TABLE Attendance ADD CONSTRAINT Classes_Attendance
   FOREIGN KEY (ClassID) 
   REFERENCES Classes (ClassID);


-- Reference: Classes_Modules (table: Classes)
ALTER TABLE Classes ADD CONSTRAINT Classes_Modules
   FOREIGN KEY (ModuleID)
   REFERENCES Modules (ModuleID);


-- Reference: Classes_Practises (table: Classes)
ALTER TABLE Classes ADD CONSTRAINT Classes_Practises
   FOREIGN KEY (PractiseID)
   REFERENCES Practises (PractiseID);


-- Reference: Classes_Subjects (table: Classes)
ALTER TABLE Classes ADD CONSTRAINT Classes_Subjects
   FOREIGN KEY (SubjectID)
   REFERENCES Subjects (SubjectID);


-- Reference: Classes_Teachers (table: Classes)
ALTER TABLE Classes ADD CONSTRAINT Classes_Teachers
   FOREIGN KEY (TeacherID)
   REFERENCES Teachers (TeacherID);


-- Reference: EducationalPrograms_ Translators (table: EducationalPrograms)
ALTER TABLE EducationalPrograms ADD CONSTRAINT EducationalPrograms_Translators
   FOREIGN KEY (TranslatorID)
   REFERENCES  Translators (TranslatorID);


-- Reference: EducationalPrograms_Courses (table: EducationalPrograms)
ALTER TABLE EducationalPrograms ADD CONSTRAINT EducationalPrograms_Courses
   FOREIGN KEY (CourseID)
   REFERENCES Courses (CourseID);


-- Reference: EducationalPrograms_Studies (table: EducationalPrograms)
ALTER TABLE EducationalPrograms ADD CONSTRAINT EducationalPrograms_Studies
   FOREIGN KEY (StudiesID)
   REFERENCES Studies (StudiesID);


-- Reference: EducationalPrograms_Teachers (table: EducationalPrograms)
ALTER TABLE EducationalPrograms ADD CONSTRAINT EducationalPrograms_Teachers
   FOREIGN KEY (LecturerID)
   REFERENCES Teachers (TeacherID);


-- Reference: EducationalPrograms_Webinars (table: EducationalPrograms)
ALTER TABLE EducationalPrograms ADD CONSTRAINT EducationalPrograms_Webinars
   FOREIGN KEY (WebinarID)
   REFERENCES Webinars (WebinarID);


-- Reference: Exams_Students (table: Exams)
ALTER TABLE Exams ADD CONSTRAINT Exams_Students
   FOREIGN KEY (StudentID)
   REFERENCES Students (StudentID);


-- Reference: Exams_Studies (table: Exams)
ALTER TABLE Exams ADD CONSTRAINT Exams_Studies
   FOREIGN KEY (StudiesID)
   REFERENCES Studies (StudiesID);


-- Reference: Modules_EducationalPrograms (table: Modules)
ALTER TABLE Modules ADD CONSTRAINT Modules_EducationalPrograms
   FOREIGN KEY (ProgramID)
   REFERENCES EducationalPrograms (ProgramID);


-- Reference: OfflineClasses_Classes (table: OfflineClasses)
ALTER TABLE OfflineClasses ADD CONSTRAINT OfflineClasses_Classes
   FOREIGN KEY (ClassID)
   REFERENCES Classes (ClassID);


-- Reference: OnlineClasses_Classes (table: OnlineClasses)
ALTER TABLE OnlineClasses ADD CONSTRAINT OnlineClasses_Classes
   FOREIGN KEY (ClassID)
   REFERENCES Classes (ClassID);


-- Reference: OrderClasses_Classes (table: RegisteredClasses)
ALTER TABLE RegisteredClasses ADD CONSTRAINT OrderClasses_Classes
   FOREIGN KEY (ClassID)
   REFERENCES Classes (ClassID);


-- Reference: OrderClasses_Orders (table: RegisteredClasses)
ALTER TABLE RegisteredClasses ADD CONSTRAINT OrderClasses_Orders
   FOREIGN KEY (OrderID)
   REFERENCES Orders (OrderID);

-- Reference: OrderDetails_EducationalPrograms (table: RegisteredPrograms)
ALTER TABLE RegisteredPrograms ADD CONSTRAINT OrderDetails_EducationalPrograms
   FOREIGN KEY (ProgramID)
   REFERENCES EducationalPrograms (ProgramID);


-- Reference: OrderDetails_Orders (table: RegisteredPrograms)
ALTER TABLE RegisteredPrograms ADD CONSTRAINT OrderDetails_Orders
   FOREIGN KEY (OrderID)
   REFERENCES Orders (OrderID);


-- Reference: Orders_Students (table: Orders)
ALTER TABLE Orders ADD CONSTRAINT Orders_Students
   FOREIGN KEY (StudentID)
   REFERENCES Students (StudentID);


-- Reference: Payments_Orders (table: Payments)
ALTER TABLE Payments ADD CONSTRAINT Payments_Orders
   FOREIGN KEY (OrderID)
   REFERENCES Orders (OrderID);


-- Reference: Practises_Studies (table: Practises)
ALTER TABLE Practises ADD CONSTRAINT Practises_Studies
   FOREIGN KEY (StudiesID)
   REFERENCES Studies (StudiesID);


-- Reference: Students_Countries (table: Students)
ALTER TABLE Students ADD CONSTRAINT Students_Countries
   FOREIGN KEY (CountryID)
   REFERENCES Countries (CountryID);


-- Reference: Subjects_SubjectCategories (table: Subjects)
ALTER TABLE Subjects ADD CONSTRAINT Subjects_SubjectCategories
   FOREIGN KEY (CategoryID)
   REFERENCES SubjectCategories (CategoryID);


-- Reference: Teachers_Countries (table: Teachers)
ALTER TABLE Teachers ADD CONSTRAINT Teachers_Countries
   FOREIGN KEY (CountryID)
   REFERENCES Countries (CountryID);


-- Reference: Webinars_Classes (table: Webinars)
ALTER TABLE Webinars ADD CONSTRAINT Webinars_Classes
   FOREIGN KEY (ClassID)
   REFERENCES Classes (ClassID);
```

<div style="page-break-after: always;"></div>

## 4. **Views**

### **1. Financial Reports – revenue summary for each webinar/course/study program**
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



### **2. "Debtors" list – individuals who have used the services but have not made the payments**
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

### **3. General report on the number of registered participants for upcoming events – future classes (with information on whether the classes are in-person or remote)**
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


### **4. General report on the number of registered participants for upcoming educational programs, along with their start dates**
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

### **5. General report on the list of individuals registered for in-person classes within an educational program**
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

### **6. Attendance list for each training session, including date, first name, last name, and information on whether the participant was present or absent**
```sql

CREATE VIEW AttendanceAllClasses as 
SELECT C.ClassID, CONCAT(year(C.StartTime), '-', month(C.StartTime), '-', day(C.StartTime)) as Date, Students.FirstName + ' ' + Students.LastName as Student, A.Present
FROM Classes C
LEFT OUTER JOIN Attendance A on C.ClassID = A.ClassID
LEFT OUTER JOIN Students on Students.StudentID = A.ParticipantID
```
![AttendanceAllClasses](img/AttendanceAllClasses.png)

<div style="page-break-after: always;"></div>

### **7. Bilocation report: List of conflicting classes along with student information, class ID, and conflicting schedules**
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

### **8. Number of participants for each completed event**
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

### **9. Data for each conducted exam, including the grade, student ID, study program ID, educational program ID, and the start & end dates of the respective studies**
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

### **10. List of subjects conducted within the modules of specific study programs, along with information about the category and the teacher for each subject**
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

### **11. List of subjects conducted within the modules of specific courses, along with information about the category and the teacher for each subject**
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

### **12. List of upcoming webinars, displaying the subject name, the first and last name of the teacher conducting the webinar, the duration of the webinar, and the access period to the webinar**
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

### **13. List of educational programs that students are enrolled in, along with the order number in which the program was purchased, the start and end dates of the program, and completion status**
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

### **14. List of individuals enrolled in the specific session "externally". The view displays the student's first and last name, the order number in which the session was purchased, the session number, access status, the subject name for the session, the module name under which the session is conducted, and the name of the practice (if the session is a practice)**
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



<div style="page-break-after: always;"></div>

## 5. **Procedures**

### **1. Adding a New Student**
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
### **2. Deleting a Student's Data**
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

### **3. Adding a New Course**
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

### **4. Updating Student Data (only email or country)**
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

### **5. Adding a new teacher**
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

### **6. Marking the make-up of student absences for classes**
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

### **7. Adding an order**

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
### **8. Adding classes to an order**
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

### **9. Adding an educational program to an order**
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

### **10. Adding a new online class**
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

### **11. Adding a new in-person class**
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

### **12. Adding a new webinar**
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


### **13. Adding a new course**
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


### **14. Adding new studies**
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

### **15. Changing the details of an educational program**
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

### **16. Adding payment to a submitted order**
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

### **17. Adding new offline classes within a study program**
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
### **18. Procedure to update a student's exam grade for specific studies. If the grade falls within the range of 50%-100%, the completion status of the corresponding program in RegisteredPrograms is updated via trigger #2**
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

### **19. "Explicit" change of access status to an educational program, which can only be set explicitly by the school director**
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

### **20. "Explicit" change of access status to individual classes, which can only be set explicitly by the school director**
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
<div style="page-break-after: always;"></div>

## 4. **Functions**

### **1. Calculating the average grade for a student**
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

### **2. Number of students present at the class**
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

### **3. Calculating the number of days remaining until the completion of the educational program**
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

### **4. Calculating the total amount for all programs in a given order**

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

### **5. Calculating the sum of registration fees for programs in a given order**

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

### **6. Calculating the total amount spent by a student on educational programs for a specific time period**
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

### **7. Displaying the class schedule for a specific day for a specific student**
```sql
CREATE FUNCTION ScheduleForStudent(@StudentID int, @day DATE)
    RETURNS TABLE
        AS
        RETURN
        SELECT Student, ModuleName, SubjectName, Teacher, StartTime, EndTime, RoomNumber
        FROM OfflineParticipantsList as ofp
        WHERE @StudentID = ofp.StudentID and @day = cast(StartTime as date) and Access = 'true'
```

### **8. Calculating the number of programs ordered by students in a given year**
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

### **9. Displaying currently ongoing synchronous online classes**
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

### **10. Calculating the average grade for individual sessions (only if a grade has been assigned to each participant of the session)**
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

### **11. Function checking the minimum possible number of participants for a session within a study module (if the session is added to the studies)**
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

### **12. Displaying the list of people enrolled in a specific offline event within a study program**
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

### **13. Displaying the list of people enrolled in a specific program**
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

### **14. Displaying the list of studies a student is enrolled in and has access to. This function is used in procedure #18 and trigger #2**
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

### **15. Displaying the list of webinars a student is enrolled in and has access to**
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

### **16. Displaying the list of courses a student is enrolled in and has access to**
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

<div style="page-break-after: always;"></div>

## 6. **Triggers**
### **1. Updating the payment status of an order after a successful transaction in the Payments table, and granting access after the payment of the enrollment fees**
```sql
ALTER TRIGGER trg_UpdateOrderStatus
ON Payments
AFTER UPDATE
AS
BEGIN
  SET NOCOUNT ON;
  IF UPDATE(Status)
  BEGIN
      IF (SELECT Status FROM INSERTED) = 1
      BEGIN
          DECLARE @OrderID INT;
          DECLARE @PaymentAmount MONEY;

          SELECT @OrderID = i.OrderID
          FROM INSERTED i;

          SELECT @PaymentAmount = i.Amount
          FROM INSERTED i;

          IF(@PaymentAmount = dbo.CalculateFullPriceForOrder(@OrderID))
          BEGIN
              UPDATE Orders
              SET OrderStatus = 'FULL PAID'
              WHERE OrderID = @OrderID

              UPDATE RegisteredPrograms
              SET Access = 1
              WHERE OrderID = @OrderID

              UPDATE RegisteredClasses
              SET Access = 1
              WHERE OrderID = @OrderID
          END

          ELSE
          BEGIN
              IF(@PaymentAmount = dbo.CalculateFullPriceForOrder(@OrderID))
              BEGIN
                  UPDATE Orders
                  SET OrderStatus = 'ENTRY PAID'
                  WHERE OrderID = @OrderID
                 
                  UPDATE RegisteredPrograms
                  SET Access = 1
                  WHERE OrderID = @OrderID

                  UPDATE RegisteredClasses
                  SET Access = 1
                  WHERE OrderID = @OrderID
              END
              ELSE
              BEGIN
                  UPDATE Orders
                  SET OrderStatus = 'NOT PAID'
                  WHERE OrderID = @OrderID
              END
          END
      END
  END
END;
```
<div style="page-break-after: always;"></div>

### **2. Setting the completion status of a specific student's studies upon updating/adding their exam grade, which must fall within the range of 50% to 100%**
```sql
CREATE TRIGGER PassStudies
ON Exams
AFTER UPDATE
AS
BEGIN
   SET NOCOUNT ON
   IF (select Mark from inserted) >= 50
   BEGIN
       DECLARE @StudiesID INT
       DECLARE @StudentID INT
       SELECT @StudiesID = StudiesID FROM INSERTED
       SELECT @StudentID = StudentID FROM INSERTED
       UPDATE RegisteredPrograms set Passed = 'true'
       WHERE RegisteredPrograms.RegisteredProgramID in
         (select ss.RegisteredProgramID from StudentStudies(@StudentID) as ss where ss.StudiesID = @StudiesID)
   END
END
```


### **3. Trigger that prevents enrollment in classes that are already full**
```sql
CREATE TRIGGER ClassesParticipantsAmountCheck
ON RegisteredClasses
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON
    BEGIN
        DECLARE @ClassID INT
        DECLARE @MaxParticipants INT
        SELECT @ClassID = ClassID FROM INSERTED
        SELECT @MaxParticipants = MaxParticipants from OfflineClasses where ClassID = @ClassID
        IF (SELECT COUNT(*) FROM AllClassParticipants(@ClassID) WHERE Access = 'true') > @MaxParticipants
        BEGIN
            THROW 52313, N'Participants number over MaxParticipants number for this Classes', 1;
        END
    END
END
```

### **4. Trigger that prevents registration for a given educational program if it is a study program and the maximum number of participants has been exceeded**
```sql
CREATE TRIGGER StudiesParticipantsAmountCheck
ON RegisteredPrograms
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON
    BEGIN
        DECLARE @ProgramID INT
        DECLARE @MaxParticipants INT
        SELECT @ProgramID = ProgramID FROM INSERTED
        IF (SELECT StudiesID from EducationalPrograms where ProgramID = @ProgramID) IS NOT NULL
            SELECT @MaxParticipants = MaxParticipants from EducationalPrograms as ep 
                inner join Studies as st on ep.StudiesID = st.StudiesID and ProgramID = @ProgramID
            IF (SELECT COUNT(*) FROM AllProgramParticipants(@ProgramID) WHERE Access = 'true') > @MaxParticipants
            BEGIN
                THROW 52313, N'Participants number over MaxParticipants number for this Studies', 1;
            END
    END
END
```
<div style="page-break-after: always;"></div>

## 7. **Indexes**
```sql
CREATE INDEX idx_student_firstname_lastname
ON Students (FirstName, LastName);

CREATE INDEX idx_teacher_firstname_lastname
ON Teachers (FirstName, LastName);

CREATE INDEX idx_translator_firstname_lastname
ON Translators (FirstName, LastName);

CREATE INDEX idx_orders_studentID
ON Orders (StudentID);

CREATE INDEX idx_registeredPrograms_orderID
ON RegisteredPrograms (OrderID);

CREATE INDEX idx_registeredPrograms_programID
ON RegisteredPrograms (ProgramID);

CREATE INDEX idx_registeredClasses_orderID
ON RegisteredClasses (OrderID);

CREATE INDEX idx_registeredClasses_classID
ON RegisteredClasses (ClassID);

CREATE INDEX idx_program_dates
ON EducationalPrograms (ProgramStart, ProgramEnd);

CREATE INDEX idx_subjects_categoryID
ON Subjects (CategoryID);

CREATE INDEX idx_classes_time
ON Classes (StartTime, EndTime);

CREATE INDEX idx_OnlineClasses_ClassID
ON OnlineClasses (ClassID)

CREATE INDEX idx_OfflineClasses_ClassID
ON OfflineClasses (ClassID)

CREATE INDEX idx_exam_studentID
ON Exams (StudentID);

CREATE INDEX idx_modules_programID
ON Modules (ProgramID);

CREATE INDEX idx_practises_studiesID
ON Practises (StudiesID);

CREATE INDEX idx_webinars_classID
ON Webinars (ClassID);

CREATE INDEX idx_classes_teacherID
ON Classes (TeacherID);

CREATE INDEX idx_classes_subjectID
ON Classes (SubjectID);

CREATE INDEX idx_attendance_classID
ON Attendance (ClassID);

CREATE INDEX idx_attendance_participantID
ON Attendance (ParticipantID);

CREATE INDEX idx_Classes_ModuleID ON Classes (ModuleID)

CREATE INDEX idx_Payments_OrderID ON Payments (OrderID)
```

## 8. **Roles**

### **Administrator**
```sql
CREATE ROLE admin
GRANT ALL PRIVILEGES ON u_smyka.dbo to admin
```

### **School Director**
```sql
CREATE ROLE director

GRANT SELECT ON WebinarsRevenue to director
GRANT SELECT ON CoursesRevenue to director
GRANT SELECT ON StudiesRevenue to director
GRANT SELECT ON Debtors to director
GRANT SELECT ON NumOfInterestedInFutureClasses to director
GRANT SELECT ON NumOfInterestedInFutureEducationalPrograms to director
GRANT SELECT ON OfflineParticipantsList to director
GRANT SELECT ON AttendanceAllClasses to director
GRANT SELECT ON BilocationsList to director
GRANT SELECT ON NumberOfParticipations to director
GRANT SELECT ON ExamDetails to director
GRANT SELECT ON StudiesSubjectsInfo to director
GRANT SELECT ON CoursesSubjectsInfo to director
GRANT SELECT ON WebinarsInfo to director
GRANT SELECT ON StudentsPrograms to director
GRANT SELECT ON StudentsOuterClasses to director

GRANT EXECUTE ON AddStudent to director
GRANT EXECUTE ON DeleteStudent to director
GRANT EXECUTE ON AddCourse to director
GRANT EXECUTE ON ChangeStudentData to director
GRANT EXECUTE ON AddTeacher to director
GRANT EXECUTE ON AddOnlineClass to director
GRANT EXECUTE ON AddOfflineClass to director
GRANT EXECUTE ON RedoAttendance to director
GRANT EXECUTE ON AddStudies to director
GRANT EXECUTE ON UpdateEducationalProgram to director
GRANT EXECUTE ON AddStudiesOfflineClasses to director
GRANT EXECUTE ON SetProgramAccess to director
GRANT EXECUTE ON SetClassesAccess to director
GRANT EXECUTE ON SetExamMark to director

GRANT EXECUTE ON CalculateAverageGradeForStudent to director
GRANT EXECUTE ON GetClassAttendanceCount to director
GRANT EXECUTE ON DaysRemainingInProgram to director
GRANT EXECUTE ON CalculateFullPriceForOrder to director
GRANT EXECUTE ON CalculateEntryPriceForOrder to director
GRANT EXECUTE ON OrdersProgramsAmount to director
GRANT EXECUTE ON AverageMarkOnClass to director
GRANT EXECUTE ON CalculateMinClassParticipantsForStudies to director

GRANT SELECT ON ScheduleForStudent to director
GRANT SELECT ON LiveOnlineSynchClasses to director
GRANT SELECT ON StudentStudies to director
GRANT SELECT ON StudentWebinars to director
GRANT SELECT ON StudentCourses to director
GRANT SELECT ON AllClassParticipants to director
GRANT SELECT ON AllProgramParticipants to director
```
<div style="page-break-after: always;"></div>

### **System Administrator (moderator)**
```sql
CREATE ROLE moderator

GRANT SELECT ON WebinarsRevenue to moderator
GRANT SELECT ON CoursesRevenue to moderator
GRANT SELECT ON StudiesRevenue to moderator
GRANT SELECT ON Debtors to moderator
GRANT SELECT ON NumOfInterestedInFutureClasses to moderator
GRANT SELECT ON NumOfInterestedInFutureEducationalPrograms to moderator
GRANT SELECT ON OfflineParticipantsList to moderator
GRANT SELECT ON AttendanceAllClasses to moderator
GRANT SELECT ON BilocationsList to moderator
GRANT SELECT ON NumberOfParticipations to moderator
GRANT SELECT ON ExamDetails to moderator
GRANT SELECT ON StudiesSubjectsInfo to moderator
GRANT SELECT ON CoursesSubjectsInfo to moderator
GRANT SELECT ON WebinarsInfo to moderator
GRANT SELECT ON StudiesSubjectsInfo to moderator
GRANT SELECT ON StudentsPrograms to moderator
GRANT SELECT ON StudentsOuterClasses to moderator

GRANT EXECUTE ON AddStudent to moderator
GRANT EXECUTE ON DeleteStudent to moderator
GRANT EXECUTE ON AddCourse to moderator
GRANT EXECUTE ON ChangeStudentData to moderator
GRANT EXECUTE ON AddTeacher to moderator
GRANT EXECUTE ON AddOnlineClass to moderator
GRANT EXECUTE ON AddOfflineClass to moderator
GRANT EXECUTE ON AddWebinar to moderator
GRANT EXECUTE ON RedoAttendance to moderator
GRANT EXECUTE ON AddStudies to moderator
GRANT EXECUTE ON UpdateEducationalProgram to moderator
GRANT EXECUTE ON AddStudiesOfflineClasses to moderator

GRANT EXECUTE ON CalculateAverageGradeForStudent to moderator
GRANT EXECUTE ON GetClassAttendanceCount to moderator
GRANT EXECUTE ON DaysRemainingInProgram to moderator
GRANT EXECUTE ON CalculateFullPriceForOrder to moderator
GRANT EXECUTE ON CalculateEntryPriceForOrder to moderator
GRANT EXECUTE ON OrdersProgramsAmount to moderator
GRANT EXECUTE ON AverageMarkOnClass to moderator
GRANT EXECUTE ON CalculateMinClassParticipantsForStudies to moderator

GRANT SELECT ON ScheduleForStudent to moderator
GRANT SELECT ON LiveOnlineSynchClasses to moderator
GRANT SELECT ON StudentStudies to moderator
GRANT SELECT ON StudentWebinars to moderator
GRANT SELECT ON StudentCourses to moderator
GRANT SELECT ON AllClassParticipants to moderator
GRANT SELECT ON AllProgramParticipants to moderator
```
<div style="page-break-after: always;"></div>

### **Educator (teacher & translator)**
```sql
CREATE ROLE educator

GRANT SELECT ON OfflineParticipantsList to educator
GRANT SELECT ON AttendanceAllClasses to educator
GRANT SELECT ON NumberOfParticipations to educator
GRANT SELECT ON ExamDetails to educator
GRANT SELECT ON StudiesSubjectsInfo to educator
GRANT SELECT ON CoursesSubjectsInfo to educator
GRANT SELECT ON WebinarsInfo to educator
GRANT SELECT ON StudentsOuterClasses to educator

GRANT EXECUTE ON RedoAttendance to educator

GRANT EXECUTE ON CalculateAverageGradeForStudent to educator
GRANT EXECUTE ON GetClassAttendanceCount to educator
GRANT EXECUTE ON DaysRemainingInProgram to educator
GRANT EXECUTE ON AverageMarkOnClass to educator

GRANT SELECT ON LiveOnlineSynchClasses to educator
GRANT SELECT ON AllProgramParticipants to educator
```

### **Student**
```sql
CREATE ROLE student

GRANT SELECT ON OfflineParticipantsList to student
GRANT SELECT ON BilocationsList to student
GRANT SELECT ON ExamDetails to student
GRANT SELECT ON StudiesSubjectsInfo to student
GRANT SELECT ON CoursesSubjectsInfo to student
GRANT SELECT ON WebinarsInfo to student
GRANT SELECT ON StudentsPrograms to student

GRANT EXECUTE ON AddOrder to student
GRANT EXECUTE ON RegisterClass to student
GRANT EXECUTE ON RegisterProgram to student
GRANT EXECUTE ON AddPayment to student

GRANT EXECUTE ON CalculateAverageGradeForStudent to student
GRANT EXECUTE ON DaysRemainingInProgram to student
GRANT EXECUTE ON AverageMarkOnClass to student

GRANT SELECT ON ScheduleForStudent to student
GRANT SELECT ON StudentStudies to student
GRANT SELECT ON StudentWebinars to student
GRANT SELECT ON StudentCourses to student
GRANT SELECT ON AllClassParticipants to student
GRANT SELECT ON AllProgramParticipants to student
```


## 9. **Data Generator**
We used Python to write the data generator. For communication with the database, we used the pyodbc library, and for generating random values, we used the Faker library.