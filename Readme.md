# **Podstawy baz danych Projekt**


Bernard Gawor, Olgierd Smyka, Stas Kochevenko
## 1. **Funkcjonalność systemu**

### **Użytkownicy**

Klient bez konta:

- Wyświetlanie oferowanych usług
- Utworzenie konta indywidualnego bądź firmowego

Klient indywidualny:

- Wyświetlanie oferowanych usług
- Możliwość zapisu na szkolenia bezpłatne lub płatne poprzez uiszczenie opłaty
- Wyświetlanie wszystkich kursów i szkoleń, na które jest zapisany oraz szczegółów ich dotyczących
- Dostęp do materiałów dydaktycznych w kursach i na studiach

Klient firmowy:

- Wyświetlanie oferowanych usług
- Możliwość zapisu na szkolenia bezpłatne lub płatne poprzez uiszczenie opłaty (indywidualnie bądź na firmę)
- Wyświetlanie wszystkich kursów i szkoleń, na które jest zapisany oraz szczegółów ich dotyczących
- Dostęp do materiałów dydaktycznych w kursach i na studiach.
- Wyświetlanie faktur za szkolenia

Pracownicy:

- Wyświetlanie wszystkich kursów i szkoleń, które prowadzi oraz szczegółów ich dotyczących oraz możliwość ich edycji
- Dostęp do materiałów dydaktycznych w kursach i na studiach oraz możliwość ich edycji

Administrator (dyrektor szkoły):

- Aktualizacja oferowanych usług
- Wystawianie faktur
- Edycja listy pracowników
- Zmiana uprawnień użytkownika
- Dostęp do funkcji systemowych oraz możliwość ich udostępnienia

## **System**

- Raporty finansowe – zestawienie przychodów dla każdego webinaru/kursu/studium
- Lista „dłużników” – osoby, które skorzystały z usług, ale nie uiściły opłat.
- Ogólny raport dotyczący liczby zapisanych osób na przyszłe wydarzenia (z informacją, czy wydarzenie jest stacjonarnie, czy zdalnie)
- Ogólny raport dotyczący frekwencji na zakończonych już wydarzeniach
- Lista obecności dla każdego szkolenia z datą, imieniem, nazwiskiem i informacją czy uczestnik był obecny, czy nie
- Raport bilokacji: lista osób, które są zapisane na co najmniej dwa przyszłe szkolenia, które ze sobą kolidują czasowo

## **Specyfikacje**

- Kursy i studia mogą odbywać się: online, stacjonarnie, hybrydowo
- Stacjonarne zajęcia kursów i studiów posiadają limit miejsc
- Webinary udostępniane są uczestnikom na okres 30 dni
- Zaliczenie kursu wymaga zaliczenia min. 80% modułów
- W przypadku studiów wymagane jest zaliczenie praktyk oraz frekwencja na poziomie minimum 80%, przy czym nieobecności mogą zostać odrobione poprzez uczestnictwo w zajęciach lub kursie komercyjnym o zbliżonej tematyce
- Tematyka programów studiów nie może być modyfikowana po ich rozpoczęciu
- Praktyki trwają 14 dni – wymagana jest tu 100% frekwencja
- Możliwość zapisania się na pojedyncze spotkania bez konieczności udziału w całym studium, przy tym cena jest inna
- Administrator ma możliwość zapisu klientów na nieopłacone szkolenia
- Uczestnictwo w kursie wymaga wpłacenia zaliczki przy zapisie, oraz dopłaty całości kwoty najpóźniej 3 dni przed rozpoczęciem kursu
- Uczestnictwo w studium wymaga uiszczenia wpisowego oraz uiszczenia opłaty za dany zjazd najpóźniej 3 dni przed jego rozpoczęciem
- Szkolenia mogą być prowadzone w różnych ustalonych językach
- Wszystkie zajęcia online odbywają się na zewnętrznej platformie chmurowej
- System płatności jest dostarczany przez zewnętrzną firmę

<div style="page-break-after: always;"></div>

## **2.Schemat Bazy Danych**
![dbschema](img/Projekt-2023-12-22.png)

```sql
-- tables
-- Table:  Translators
CREATE TABLE  Translators (
    TranslatorID int  NOT NULL,
    FirstName varchar(20)  NOT NULL,
    LastName varchar(20)  NOT NULL,
    CountryID int  NOT NULL,
    CONSTRAINT Translators_pk PRIMARY KEY  (TranslatorID)
);

-- Table: Attendance
CREATE TABLE Attendance (
    AttendanceID int  NOT NULL,
    ClassID int  NOT NULL,
    Present bit  NOT NULL,
    ParticipantID int  NOT NULL,
    Redone bit  NOT NULL,
    CONSTRAINT Attendance_pk PRIMARY KEY  (AttendanceID)
);

-- Table: Classes
CREATE TABLE Classes (
    ClassID int  NOT NULL,
    TeacherID int  NOT NULL,
    SubjectID int  NOT NULL,
    StartTime datetime  NOT NULL,
    EndTime datetime  NOT NULL,
    ClassPrice money  NOT NULL,
    CONSTRAINT Classes_pk PRIMARY KEY  (ClassID)
);

-- Table: ClassesRegistrations
CREATE TABLE ClassesRegistrations (
    RegistrationID int  NOT NULL,
    StudentID int  NOT NULL,
    OrderID int  NOT NULL,
    ClassID int  NOT NULL,
    CONSTRAINT ClassesRegistrations_pk PRIMARY KEY  (RegistrationID)
);

-- Table: Countries
CREATE TABLE Countries (
    CountryID int  NOT NULL,
    CountryName varchar(40) NOT NULL,
    CONSTRAINT Countries_pk PRIMARY KEY  (CountryID)
);

-- Table: Courses
CREATE TABLE Courses (
    CourseID int  NOT NULL,
    Place varchar(40)  NOT NULL,
    Advance money  NOT NULL,
    CONSTRAINT Courses_pk PRIMARY KEY  (CourseID)
);

-- Table: EducationalPrograms
CREATE TABLE EducationalPrograms (
    ProgramID int  NOT NULL,
    ProgramName int  NOT NULL,
    StudiesID int  NULL,
    WebinarID int  NULL,
    CourseID int  NULL,
    Language varchar(20)  NOT NULL,
    ProgramStart date  NOT NULL,
    ProgramEnd date  NOT NULL,
    ProgramPrice money  NOT NULL,
    LecturerID int  NOT NULL,
    TranslatorID int  NULL,
    CONSTRAINT EducationalPrograms_pk PRIMARY KEY  (ProgramID)
);

-- Table: Exams
CREATE TABLE Exams (
    ExamID int  NOT NULL,
    StudiesID int  NOT NULL,
    StudentID int  NOT NULL,
    Mark int  NOT NULL,
    CONSTRAINT Exams_pk PRIMARY KEY  (ExamID)
);

-- Table: ModuleDetails
CREATE TABLE ModuleDetails (
    ModuleID int  NOT NULL,
    ClassID int  NOT NULL,
    CONSTRAINT ModuleDetails_pk PRIMARY KEY  (ModuleID)
);

-- Table: Modules
CREATE TABLE Modules (
    ModuleID int  NOT NULL,
    CourseID int  NOT NULL,
    CONSTRAINT Modules_pk PRIMARY KEY  (ModuleID)
);

-- Table: OfflineClasses
CREATE TABLE OfflineClasses (
    OfflineClassID int  NOT NULL,
    ClassID int  NOT NULL,
    RoomNumber int  NOT NULL,
    MinParticipants int  NOT NULL,
    Mark int  NULL,
    CONSTRAINT OfflineClasses_pk PRIMARY KEY  (OfflineClassID)
);

-- Table: OnlineClasses
CREATE TABLE OnlineClasses (
    OnlineClassID int  NOT NULL,
    ClassID int  NOT NULL,
    Link varchar(255)  NOT NULL,
    Synch bit  NOT NULL,
    CONSTRAINT OnlineClasses_pk PRIMARY KEY  (OnlineClassID)
);

-- Table: OrderedClasses
CREATE TABLE OrderedClasses (
    OrderedClassID int  NOT NULL,
    OrderID int  NOT NULL,
    ClassID int  NOT NULL,
    CONSTRAINT OrderedClasses_pk PRIMARY KEY  (OrderedClassID)
);

-- Table: OrderedPrograms
CREATE TABLE OrderedPrograms (
    OrderedProgramID int  NOT NULL,
    OrderID int  NOT NULL,
    ProgramID int  NOT NULL,
    CONSTRAINT OrderedPrograms_pk PRIMARY KEY  (OrderedProgramID)
);

-- Table: Orders
CREATE TABLE Orders (
    OrderID int  NOT NULL,
    StudentID int  NOT NULL,
    OrderDate datetime  NOT NULL,
    CONSTRAINT Orders_pk PRIMARY KEY  (OrderID)
);

-- Table: Payments
CREATE TABLE Payments (
    PaymentID int  NOT NULL,
    OrderID int  NOT NULL,
    Paid money  NOT NULL,
    Date date  NOT NULL,
    Status bit  NOT NULL,
    CONSTRAINT Payments_pk PRIMARY KEY  (PaymentID)
);

-- Table: PractiseDetails
CREATE TABLE PractiseDetails (
    PractiseID int  NOT NULL,
    ClassID int  NOT NULL,
    CONSTRAINT PractiseDetails_pk PRIMARY KEY  (PractiseID)
);

-- Table: Practises
CREATE TABLE Practises (
    PractiseID int  NOT NULL,
    StudiesID int  NOT NULL,
    CONSTRAINT Practices_pk PRIMARY KEY  (PractiseID)
);

-- Table: ProgramsRegistrations
CREATE TABLE ProgramsRegistrations (
    RegistrationID int  NOT NULL,
    StudentID int  NOT NULL,
    OrderID int  NOT NULL,
    Passed bit  NOT NULL,
    ToPay money  NOT NULL,
    CertificateLink varchar(40)  NULL,
    ProgramID int  NOT NULL,
    Exception bit  NOT NULL,
    CONSTRAINT ProgramsRegistrations_pk PRIMARY KEY  (RegistrationID)
);

-- Table: Students
CREATE TABLE Students (
    StudentID int  NOT NULL,
    FirstName varchar(20)  NOT NULL,
    LastName varchar(20)  NOT NULL,
    CountryID int  NOT NULL,
    Email varchar(40)  NOT NULL,
    CONSTRAINT Students_pk PRIMARY KEY  (StudentID)
);

-- Table: Studies
CREATE TABLE Studies (
    StudiesID int  NOT NULL,
    Syllabus varchar(255)  NOT NULL,
    Place varchar(40)  NOT NULL,
    MinParticipants int  NOT NULL,
    EntryFee money  NOT NULL,
    CONSTRAINT Studies_pk PRIMARY KEY  (StudiesID)
);

-- Table: SubjectCategories
CREATE TABLE SubjectCategories (
    CategoryID int  NOT NULL,
    CategoryName varchar(100)  NOT NULL,
    Description varchar(40)  NOT NULL,
    CONSTRAINT SubjectCategories_pk PRIMARY KEY  (CategoryID)
);

-- Table: Subjects
CREATE TABLE Subjects (
    SubjectID int  NOT NULL,
    CategoryID int  NOT NULL,
    Description varchar(100)  NOT NULL,
    SubjectName varchar(40)  NOT NULL,
    CONSTRAINT Subjects_pk PRIMARY KEY  (SubjectID)
);

```