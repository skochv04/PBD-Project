# Indexes

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