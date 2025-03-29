# Roles

## Administrator
```sql
CREATE ROLE admin
GRANT ALL PRIVILEGES ON u_smyka.dbo to admin
```

## School Director
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

## System Administrator (moderator)
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

## Educator (teacher & translator)
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

## Student
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