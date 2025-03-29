# Triggers
## 1. Updating the payment status of an order after a successful transaction in the Payments table, and granting access after the payment of the enrollment fees
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

## 2. Setting the completion status of a specific student's studies upon updating/adding their exam grade, which must fall within the range of 50% to 100%
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


## 3. Trigger that prevents enrollment in classes that are already full
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

## 4. Trigger that prevents registration for a given educational program if it is a study program and the maximum number of participants has been exceeded
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