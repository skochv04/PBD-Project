
## **Tabele**

Oferowane przez firmę usługi (różnego rodzaju kursy i szkolenia) łączy EducationalPrograms. Każdy rekord przedstawia albo studia (Studies), albo kurs (Courses) albo webinar (Webinars). Spis wszystkich poszczególnych zajęć (spotkań) znajduje się w tabeli Classes. Spotkania mogą być stacjonarne (OfflineClasses) lub niestacjonarne (OnlineClasses).
Kursy (Courses) składają się z modułów (Modules). Pojedyncze zajęcia tych modułów mogą być prowadzone stacjonarnie lub niestacjonarnie.
Studia podobnie do kursów składają się z modułów (Modules), oraz posiadają praktyki (Practises).


Studenci mogą składać zamówienia (Orders) i przeglądać listę programów (RegisteredPrograms) oraz pojedynczych spotkań (RegisteredClasses), na które są zapisane.

```sql
-- tables
-- Table:  Translators
CREATE TABLE  Translators (
   TranslatorID int  NOT NULL,
   FirstName varchar(20)  NOT NULL,
   LastName varchar(20)  NOT NULL,
   CountryID int  NOT NULL,
   CONSTRAINT Translator_pk PRIMARY KEY  (TranslatorID)
);


-- Table: Attendance
—- Zawiera informacje dotyczące obecności konkretnych studentów z tabeli Students na zajęciach z tabeli Classes
CREATE TABLE Attendance (
   AttendanceID int  NOT NULL,
   ClassID int  NOT NULL,
   Present bit  NOT NULL DEFAULT 0,
   ParticipantID int  NOT NULL,
   Redone bit  NOT NULL DEFAULT 0,
   CONSTRAINT Attendance_pk PRIMARY KEY  (AttendanceID)
);


-- Table: Classes
— Pojedyncze spotkanie w ramach programu edukacyjnego (albo konkretnego modułu w przypadku kursów lub studiów), może być w formacie online lub offline
CREATE TABLE Classes (
   ClassID int  NOT NULL,
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
   CountryID int  NOT NULL,
   CountryName int  NOT NULL UNIQUE,
   CONSTRAINT Countries_pk PRIMARY KEY  (CountryID)
);


-- Table: Courses
CREATE TABLE Courses (
   CourseID int  NOT NULL,
   Place varchar(40)  NOT NULL,
   Advance money  NOT NULL,
   CHECK (Advance >= 0),
   CONSTRAINT Courses_pk PRIMARY KEY  (CourseID)
);


-- Table: EducationalPrograms
—- Zawiera szczegóły konkretnego programu edukacyjnego, którym mogą być studia z tabeli Studies, kursy z tabeli Courses lub Webinary z tabeli Webinars, w każdym rekorcie tylko jedna z trzech wartości: StudiesID, WebinarID, CourseID nie jest NULL-em.
CREATE TABLE EducationalPrograms (
   ProgramID int  NOT NULL,
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


-- Table: Exams
—- Zawiera wyniki z egzaminów dla studentów (Tabela Students) zapisanych na studia(Tabela Studies)
CREATE TABLE Exams (
   ExamID int  NOT NULL,
   StudiesID int  NOT NULL,
   StudentID int  NOT NULL,
   Mark int  NOT NULL DEFAULT 0,
   CHECK(Mark >= 0 AND Mark <= 100),
   CONSTRAINT Exams_pk PRIMARY KEY  (ExamID)
);


-- Table: Modules
—- Zbiór zajęć na określony temat, nie tożsamy z pojęciem przedmiotu (jeden moduł może zawierać zajęcia z różnych przedmiotów). Pozwalają na łączenie zajęć różnej formy kształcenia (stacjonarne, online asynchroniczne, online synchroniczne, hybrydowe).
—- Dla przykładu:
—- Moduł “Programowanie w matematyce” mógłby obejmować szereg zajęć z przedmiotów matematycznych, na których problemy rozwiązywane są przy pomocy pisanego kodu
CREATE TABLE Modules (
   ModuleID int  NOT NULL,
   ProgramID int  NOT NULL,
   ModuleName varchar(40)  NOT NULL,
   ModuleDescription varchar(100)  NOT NULL,
   CONSTRAINT Modules_pk PRIMARY KEY  (ModuleID)
);


-- Table: OfflineClasses
—- Podzbiór Classes: pojedyncze zajęcia, prowadzone w trybie offline (stacjonarnie), zawsze są podporządkowane jednemu modułu zajęć.
CREATE TABLE OfflineClasses (
   OfflineClassID int  NOT NULL,
   ClassID int  NOT NULL,
   RoomNumber int  NOT NULL,
   MinParticipants int  NOT NULL DEFAULT 0,
   Mark int  NULL,
   CHECK(Mark >= 0 AND Mark <= 100),
   CONSTRAINT OfflineClasses_pk PRIMARY KEY  (OfflineClassID)
);


-- Table: OnlineClasses
—- Podzbiór Classes: pojedyncze zajęcia, prowadzone w trybie online. Obejmują synchroniczne i asynchroniczne moduły.
CREATE TABLE OnlineClasses (
   OnlineClassID int  NOT NULL,
   ClassID int  NOT NULL,
   Link varchar(255)  NOT NULL,
   Synch bit  NOT NULL DEFAULT 0,
   CONSTRAINT OnlineClasses_pk PRIMARY KEY  (OnlineClassID)
);


-- Table: Orders
—- Lista zamówień przez Studentów. Informacja o zakupionych programach oraz pojedynczych spotkaniach znajduje się w tabelach RegisteredPrograms i RegisteredClasses odpowiednio.
CREATE TABLE Orders (
   OrderID int  NOT NULL,
   StudentID int  NOT NULL,
   OrderDate datetime  NOT NULL DEFAULT GETDATE(),
   CONSTRAINT Orders_pk PRIMARY KEY  (OrderID)
);


-- Table: Payments
—- Spis płatność dokonanych w celu częściowego lub całkowitego opłacenia zamówienia z tabeli Orders. Kolumna Status informuje czy płatność została zakończona sukcesem, natomiast kolumna SystemPaymentID zawiera link do zewnętrznego systemu płatności.
CREATE TABLE Payments (
   PaymentID int  NOT NULL,
   OrderID int  NOT NULL,
   Amount money  NOT NULL,
   Date date  NOT NULL DEFAULT GETDATE(),
   Status bit  NOT NULL DEFAULT 0,
   CHECK (Amount >= 0),
   CONSTRAINT Payments_pk PRIMARY KEY  (PaymentID)
);


-- Table: Practises
—- Każde studia mogą zawierać wiele praktyk, tabela przetrzymuje opis i identyfikator danych praktyk. W Tabeli Classes znajduje się pole PracticeID, które nie jest NULL-em w przypadku gdy dane zajęcia realizują dane praktyki.
CREATE TABLE Practises (
   PractiseID int  NOT NULL,
   StudiesID int  NOT NULL,
   PracticeName varchar(40)  NOT NULL,
   PracticeDescription varchar(255)  NOT NULL,
   CONSTRAINT Practices_pk PRIMARY KEY  (PractiseID)
);


-- Table: RegisteredClasses
—- Lista zakupionych przez studentów pojedynczych classes (zjazdów w ramach studiów) z numerami zamówienia
CREATE TABLE RegisteredClasses (
   RegisteredClassID int  NOT NULL,
   OrderID int  NOT NULL,
   ClassID int  NOT NULL,
   CONSTRAINT RegisteredClasses_pk PRIMARY KEY  (RegisteredClassID)
);


-- Table: RegisteredPrograms
—- Lista zakupionych przez studentów EducationalProgramów z numerami zamówienia
CREATE TABLE RegisteredPrograms (
   RegisteredProgramID int  NOT NULL,
   OrderID int  NOT NULL,
   ProgramID int  NOT NULL,
   Passed bit  NOT NULL DEFAULT 0,
   CertificateLink varchar(255)  NOT NULL,
   CONSTRAINT RegisteredPrograms_pk PRIMARY KEY  (RegisteredProgramID)
);


-- Table: Students
CREATE TABLE Students (
   StudentID int  NOT NULL,
   FirstName varchar(20)  NOT NULL,
   LastName varchar(20)  NOT NULL,
   CountryID int  NOT NULL,
   Email varchar(40)  NOT NULL UNIQUE,
   CONSTRAINT Students_pk PRIMARY KEY  (StudentID)
);


-- Table: Studies
CREATE TABLE Studies (
   StudiesID int  NOT NULL,
   Syllabus varchar(255)  NOT NULL,
   Place varchar(100)  NOT NULL,
   MinParticipants int  NOT NULL,
   EntryFee money  NOT NULL
   CHECK (EntryFee >= 0),
   CONSTRAINT Studies_pk PRIMARY KEY  (StudiesID)
);


-- Table: SubjectCategories
—- Zawiera kategorie różnych prowadzonych przedmiotów z tabeli Subjects
—- np. Matematyka(SubjectCategories) jest kategorią przedmiotu algebra(Subjects)
CREATE TABLE SubjectCategories (
   CategoryID int  NOT NULL,
   CategoryName varchar(40)  NOT NULL UNIQUE,
   Description varchar(255)  NOT NULL,
   CONSTRAINT SubjectCategories_pk PRIMARY KEY  (CategoryID)
);


-- Table: Subjects
CREATE TABLE Subjects (
   SubjectID int  NOT NULL,
   CategoryID int  NOT NULL,
   Description varchar(255)  NOT NULL,
   SubjectName varchar(40)  NOT NULL UNIQUE,
   CONSTRAINT Subjects_pk PRIMARY KEY  (SubjectID)
);


-- Table: Teachers
CREATE TABLE Teachers (
   TeacherID int  NOT NULL,
   FirstName varchar(15)  NOT NULL,
   LastName varchar(15)  NOT NULL,
   CountryID int  NOT NULL,
   CONSTRAINT Teachers_pk PRIMARY KEY  (TeacherID)
);


-- Table: Webinars
CREATE TABLE Webinars (
   WebinarID int  NOT NULL,
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