# Database Schema and Tables
![dbschema](../img/database-schema.png)

## Description
The services offered by the company (various courses and trainings) are connected by **EducationalPrograms**. Each record represents either a study program (**Studies**), a course (**Courses**), or a webinar (**Webinars**). A list of all classes (sessions) is stored in the **Classes** table. These sessions can be in-person (**OfflineClasses**) or online (**OnlineClasses**).

**Courses** consist of **Modules**. Classes of these modules can be conducted in-person or online.

Similar to courses, study programs consist of **Modules** and also include **Practises**.

Students can make **Orders** and view the list of programs (**RegisteredPrograms**) and classes (**RegisteredClasses**) they are enrolled in.

## Tabels
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