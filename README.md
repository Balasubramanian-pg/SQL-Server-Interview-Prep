# SQL-Server-Interview-Prep
Here’s a **concise yet comprehensive SQL Server cheat sheet** tailored for interviews, covering all critical topics with examples:

### **1. SQL Server Basics**
**Editions**:
| Edition | Use Case |
|---------|----------|
| **Enterprise** | High-end OLTP, data warehousing. |
| **Standard** | Mid-tier applications. |
| **Web** | Hosting providers. |
| **Express** | Free for small apps (10GB limit). |
| **Developer** | Full-featured for dev/test. |

**Key Tools**:
- **SSMS (SQL Server Management Studio)**: GUI for management.
- **T-SQL (Transact-SQL)**: SQL Server’s dialect.
- **SQLCMD**: Command-line utility.
- **PowerShell**: Automation with `SqlServer` module.

---

### **2. T-SQL Queries**
#### **SELECT Statements**
```sql
-- Basic SELECT
SELECT Column1, Column2 FROM TableName;

-- Aliases
SELECT Column1 AS Alias1, Column2 AS Alias2 FROM TableName;

-- WHERE Clause
SELECT * FROM Employees WHERE Salary > 50000;

-- ORDER BY
SELECT * FROM Employees ORDER BY Salary DESC;

-- DISTINCT
SELECT DISTINCT Department FROM Employees;

-- TOP (or FETCH FIRST)
SELECT TOP 10 * FROM Employees;
SELECT * FROM Employees ORDER BY Salary DESC FETCH FIRST 5 ROWS ONLY;

-- BETWEEN, IN, LIKE
SELECT * FROM Employees
WHERE Salary BETWEEN 50000 AND 100000
AND Department IN ('IT', 'HR')
AND Name LIKE 'J%';

-- NULL Handling
SELECT * FROM Employees WHERE Commission IS NOT NULL;
```

#### **Aggregations & Grouping**
```sql
-- GROUP BY + HAVING
SELECT Department, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY Department
HAVING AVG(Salary) > 60000;

-- Common Aggregates
SELECT
  COUNT(*) AS TotalEmployees,
  SUM(Salary) AS TotalSalary,
  AVG(Salary) AS AvgSalary,
  MIN(Salary) AS MinSalary,
  MAX(Salary) AS MaxSalary
FROM Employees;
```

#### **Joins**
```sql
-- INNER JOIN (default)
SELECT e.Name, d.DepartmentName
FROM Employees e
INNER JOIN Departments d ON e.DepartmentID = d.ID;

-- LEFT/RIGHT JOIN
SELECT e.Name, d.DepartmentName
FROM Employees e
LEFT JOIN Departments d ON e.DepartmentID = d.ID; -- All employees, even without dept

-- FULL OUTER JOIN
SELECT e.Name, d.DepartmentName
FROM Employees e
FULL OUTER JOIN Departments d ON e.DepartmentID = d.ID; -- All records from both tables

-- CROSS JOIN
SELECT e.Name, p.ProjectName
FROM Employees e
CROSS JOIN Projects p; -- Cartesian product

-- SELF JOIN
SELECT e1.Name AS Employee, e2.Name AS Manager
FROM Employees e1
LEFT JOIN Employees e2 ON e1.ManagerID = e2.ID;
```

#### **Subqueries**
```sql
-- Subquery in WHERE
SELECT Name FROM Employees
WHERE DepartmentID IN (SELECT ID FROM Departments WHERE Location = 'NY');

-- Subquery in FROM
SELECT e.Name, dept.AvgSalary
FROM Employees e
JOIN (SELECT DepartmentID, AVG(Salary) AS AvgSalary FROM Employees GROUP BY DepartmentID) dept
ON e.DepartmentID = dept.DepartmentID;

-- Correlated Subquery
SELECT e.Name, e.Salary
FROM Employees e
WHERE e.Salary > (SELECT AVG(Salary) FROM Employees WHERE DepartmentID = e.DepartmentID);
```

#### **Common Table Expressions (CTEs)**
```sql
-- Simple CTE
WITH HighEarners AS (
  SELECT Name, Salary
  FROM Employees
  WHERE Salary > 100000
)
SELECT * FROM HighEarners;

-- Recursive CTE (for hierarchies)
WITH EmployeeHierarchy AS (
  SELECT ID, Name, ManagerID, 0 AS Level
  FROM Employees
  WHERE ManagerID IS NULL -- Top-level
  UNION ALL
  SELECT e.ID, e.Name, e.ManagerID, eh.Level + 1
  FROM Employees e
  JOIN EmployeeHierarchy eh ON e.ManagerID = eh.ID
)
SELECT * FROM EmployeeHierarchy;
```

---

### **3. Data Modification**
#### **INSERT**
```sql
-- Single row
INSERT INTO Employees (Name, Salary, DepartmentID)
VALUES ('John Doe', 75000, 3);

-- Multiple rows
INSERT INTO Employees (Name, Salary, DepartmentID)
VALUES
  ('Jane Smith', 80000, 3),
  ('Bob Johnson', 65000, 2);

-- INSERT from SELECT
INSERT INTO EmployeesBackup
SELECT * FROM Employees WHERE DepartmentID = 1;
```

#### **UPDATE**
```sql
-- Basic UPDATE
UPDATE Employees
SET Salary = Salary * 1.1
WHERE DepartmentID = 1;

-- UPDATE with JOIN
UPDATE e
SET e.Salary = e.Salary * 1.05
FROM Employees e
JOIN Departments d ON e.DepartmentID = d.ID
WHERE d.Location = 'NY';
```

#### **DELETE**
```sql
-- Basic DELETE
DELETE FROM Employees WHERE DepartmentID = 5;

-- DELETE with JOIN
DELETE e
FROM Employees e
JOIN Departments d ON e.DepartmentID = d.ID
WHERE d.Location = 'Remote';
```

#### **MERGE (Upsert)**
```sql
MERGE INTO TargetTable AS target
USING SourceTable AS source
ON target.ID = source.ID
WHEN MATCHED THEN
  UPDATE SET target.Name = source.Name, target.Salary = source.Salary
WHEN NOT MATCHED THEN
  INSERT (ID, Name, Salary) VALUES (source.ID, source.Name, source.Salary);
```

---

### **4. Table Design & Constraints**
#### **CREATE TABLE**
```sql
CREATE TABLE Employees (
  ID INT PRIMARY KEY IDENTITY(1,1),
  Name NVARCHAR(100) NOT NULL,
  Email NVARCHAR(100) UNIQUE,
  Salary DECIMAL(10,2) CHECK (Salary > 0),
  DepartmentID INT,
  HireDate DATE DEFAULT GETDATE(),
  CONSTRAINT FK_Department FOREIGN KEY (DepartmentID) REFERENCES Departments(ID)
);
```

#### **ALTER TABLE**
```sql
-- Add column
ALTER TABLE Employees ADD Commission DECIMAL(10,2);

-- Drop column
ALTER TABLE Employees DROP COLUMN Commission;

-- Modify column
ALTER TABLE Employees ALTER COLUMN Salary DECIMAL(12,2);

-- Add constraint
ALTER TABLE Employees ADD CONSTRAINT CHK_Salary CHECK (Salary BETWEEN 30000 AND 500000);
```

#### **Indexes**
```sql
-- Create index
CREATE INDEX IX_Employees_DepartmentID ON Employees(DepartmentID);

-- Clustered index (sorts table physically)
CREATE CLUSTERED INDEX IX_Employees_Name ON Employees(Name);

-- Non-clustered index (default)
CREATE INDEX IX_Employees_Salary ON Employees(Salary);

-- Composite index
CREATE INDEX IX_Employees_Dept_Salary ON Employees(DepartmentID, Salary);

-- Included columns (covering index)
CREATE INDEX IX_Employees_Name_INCLUDE_Salary
ON Employees(Name) INCLUDE (Salary);

-- Drop index
DROP INDEX IX_Employees_Salary ON Employees;
```

---

### **5. Views**
```sql
-- Create view
CREATE VIEW vw_EmployeeDetails AS
SELECT e.Name, e.Salary, d.DepartmentName
FROM Employees e
JOIN Departments d ON e.DepartmentID = d.ID;

-- Update view (with INSTEAD OF trigger)
CREATE TRIGGER tr_vw_EmployeeDetails_UPDATE
ON vw_EmployeeDetails
INSTEAD OF UPDATE
AS
BEGIN
  UPDATE Employees
  SET Salary = i.Salary
  FROM Employees e
  JOIN inserted i ON e.Name = i.Name;
END;

-- Materialized view (indexed view)
CREATE VIEW vw_SalesSummary WITH SCHEMABINDING AS
SELECT ProductID, SUM(Quantity) AS TotalQuantity, COUNT_BIG(*) AS Count
FROM dbo.Sales
GROUP BY ProductID;
GO
CREATE UNIQUE CLUSTERED INDEX IX_vw_SalesSummary ON vw_SalesSummary(ProductID);
```

---

### **6. Stored Procedures**
```sql
-- Create procedure
CREATE PROCEDURE usp_GetEmployeesByDepartment
  @DepartmentID INT
AS
BEGIN
  SELECT * FROM Employees WHERE DepartmentID = @DepartmentID;
END;

-- Execute procedure
EXEC usp_GetEmployeesByDepartment @DepartmentID = 3;

-- Procedure with output parameter
CREATE PROCEDURE usp_GetEmployeeCount
  @DepartmentID INT,
  @Count INT OUTPUT
AS
BEGIN
  SELECT @Count = COUNT(*) FROM Employees WHERE DepartmentID = @DepartmentID;
END;
-- Execute
DECLARE @EmpCount INT;
EXEC usp_GetEmployeeCount @DepartmentID = 3, @Count = @EmpCount OUTPUT;
SELECT @EmpCount;

-- Dynamic SQL in procedure
CREATE PROCEDURE usp_DynamicQuery
  @TableName NVARCHAR(100),
  @ColumnName NVARCHAR(100)
AS
BEGIN
  DECLARE @SQL NVARCHAR(MAX);
  SET @SQL = 'SELECT ' + @ColumnName + ' FROM ' + @TableName;
  EXEC sp_executesql @SQL;
END;
```

---

### **7. Functions**
#### **Scalar Functions**
```sql
CREATE FUNCTION fn_CalculateBonus(@Salary DECIMAL(10,2))
RETURNS DECIMAL(10,2)
AS
BEGIN
  RETURN @Salary * 0.1;
END;
-- Usage
SELECT Name, Salary, dbo.fn_CalculateBonus(Salary) AS Bonus FROM Employees;
```

#### **Table-Valued Functions**
```sql
-- Inline table-valued function
CREATE FUNCTION fn_GetEmployeesByDept(@DeptID INT)
RETURNS TABLE
AS
RETURN (
  SELECT Name, Salary FROM Employees WHERE DepartmentID = @DeptID
);
-- Usage
SELECT * FROM dbo.fn_GetEmployeesByDept(3);

-- Multi-statement table-valued function
CREATE FUNCTION fn_GetTopEarners(@Count INT)
RETURNS @Result TABLE (Name NVARCHAR(100), Salary DECIMAL(10,2))
AS
BEGIN
  INSERT INTO @Result
  SELECT TOP (@Count) Name, Salary FROM Employees ORDER BY Salary DESC;
  RETURN;
END;
-- Usage
SELECT * FROM dbo.fn_GetTopEarners(5);
```

---

### **8. Triggers**
```sql
-- AFTER INSERT trigger
CREATE TRIGGER tr_AfterEmployeeInsert
ON Employees
AFTER INSERT
AS
BEGIN
  INSERT INTO EmployeeAudit (EmployeeID, Action, ChangeDate)
  SELECT ID, 'INSERT', GETDATE() FROM inserted;
END;

-- INSTEAD OF UPDATE trigger
CREATE TRIGGER tr_InsteadOfEmployeeUpdate
ON Employees
INSTEAD OF UPDATE
AS
BEGIN
  -- Custom logic (e.g., validate salary increase)
  IF UPDATE(Salary)
  BEGIN
    IF (SELECT Salary FROM inserted) > (SELECT Salary FROM deleted) * 1.2
    BEGIN
      RAISERROR('Salary increase cannot exceed 20%', 16, 1);
      RETURN;
    END
  END
  -- Proceed with update
  UPDATE e
  SET e.Name = i.Name,
      e.Salary = i.Salary
  FROM Employees e
  JOIN inserted i ON e.ID = i.ID;
END;

-- DDL Trigger (for schema changes)
CREATE TRIGGER tr_PreventTableDrops
ON DATABASE
FOR DROP_TABLE
AS
BEGIN
  PRINT 'Table drop attempts are not allowed.';
  ROLLBACK;
END;
```

---

### **9. Transactions & Locking**
```sql
-- Explicit transaction
BEGIN TRANSACTION;
BEGIN TRY
  UPDATE Accounts SET Balance = Balance - 100 WHERE AccountID = 1;
  UPDATE Accounts SET Balance = Balance + 100 WHERE AccountID = 2;
  COMMIT TRANSACTION;
END TRY
BEGIN CATCH
  ROLLBACK TRANSACTION;
  PRINT 'Transaction failed: ' + ERROR_MESSAGE();
END CATCH;

-- Isolation levels
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; -- Dirty reads
SET TRANSACTION ISOLATION LEVEL READ COMMITTED; -- Default
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- Highest isolation

-- Locking hints
SELECT * FROM Employees WITH (NOLOCK); -- Dirty read
SELECT * FROM Employees WITH (UPDLOCK); -- Update lock
SELECT * FROM Employees WITH (ROWLOCK); -- Row-level lock
```

**Deadlock Handling**:
```sql
-- Retry logic in application
BEGIN TRY
  BEGIN TRANSACTION;
  -- Operations
  COMMIT TRANSACTION;
END TRY
BEGIN CATCH
  IF ERROR_NUMBER() = 1205 -- Deadlock
  BEGIN
    WAITFOR DELAY '00:00:01'; -- Wait and retry
    -- Retry logic here
  END
  ELSE
  BEGIN
    ROLLBACK TRANSACTION;
    PRINT 'Error: ' + ERROR_MESSAGE();
  END
END CATCH;
```

---

### **10. Error Handling**
```sql
BEGIN TRY
  -- Risky operation
  SELECT 1/0; -- Divide by zero
END TRY
BEGIN CATCH
  SELECT
    ERROR_NUMBER() AS ErrorNumber,
    ERROR_MESSAGE() AS ErrorMessage,
    ERROR_SEVERITY() AS ErrorSeverity,
    ERROR_STATE() AS ErrorState,
    ERROR_PROCEDURE() AS ErrorProcedure,
    ERROR_LINE() AS ErrorLine;
END CATCH;

-- THROW (SQL Server 2012+)
BEGIN CATCH
  THROW; -- Re-throws the error
END CATCH;
```

---

### **11. Dynamic SQL**
```sql
-- Basic dynamic SQL
DECLARE @SQL NVARCHAR(MAX);
DECLARE @DeptID INT = 3;
SET @SQL = 'SELECT * FROM Employees WHERE DepartmentID = ' + CAST(@DeptID AS NVARCHAR(10));
EXEC sp_executesql @SQL;

-- With parameters (safe from SQL injection)
DECLARE @SQL NVARCHAR(MAX);
DECLARE @DeptID INT = 3;
SET @SQL = 'SELECT * FROM Employees WHERE DepartmentID = @DeptID';
EXEC sp_executesql @SQL, N'@DeptID INT', @DeptID;
```

---

### **12. XML & JSON**
#### **XML**
```sql
-- Query XML
DECLARE @XML XML = '
<Employees>
  <Employee ID="1" Name="John" Salary="75000"/>
  <Employee ID="2" Name="Jane" Salary="80000"/>
</Employees>';
SELECT
  x.value('@ID', 'INT') AS ID,
  x.value('@Name', 'NVARCHAR(100)') AS Name,
  x.value('@Salary', 'DECIMAL(10,2)') AS Salary
FROM @XML.nodes('/Employees/Employee') AS T(x);

-- FOR XML PATH
SELECT Name, Salary
FROM Employees
WHERE DepartmentID = 1
FOR XML PATH('Employee'), ROOT('Employees');
```

#### **JSON**
```sql
-- Query JSON
DECLARE @JSON NVARCHAR(MAX) = '
[
  {"ID": 1, "Name": "John", "Salary": 75000},
  {"ID": 2, "Name": "Jane", "Salary": 80000}
]';
SELECT
  JSON_VALUE(value, '$.ID') AS ID,
  JSON_VALUE(value, '$.Name') AS Name,
  JSON_VALUE(value, '$.Salary') AS Salary
FROM OPENJSON(@JSON);

-- FOR JSON PATH
SELECT Name, Salary
FROM Employees
WHERE DepartmentID = 1
FOR JSON PATH;
```

---

### **13. Performance Tuning**
#### **Execution Plans**
```sql
-- View execution plan
SET SHOWPLAN_TEXT ON; -- Text plan
SET SHOWPLAN_XML ON;  -- XML plan
GO
SELECT * FROM Employees WHERE Salary > 50000;
GO
SET SHOWPLAN_XML OFF;

-- Actual execution plan (includes runtime stats)
SELECT * FROM Employees WHERE Salary > 50000;
```

#### **Index Tuning**
```sql
-- Missing Index DMV
SELECT * FROM sys.dm_db_missing_index_details;
SELECT * FROM sys.dm_db_missing_index_group_stats;
SELECT * FROM sys.dm_db_missing_index_groups;

-- Unused indexes
SELECT * FROM sys.dm_db_index_usage_stats
WHERE user_seeks = 0 AND user_scans = 0 AND user_lookups = 0;
```

#### **Query Optimizations**
- **Avoid `SELECT *`**: Specify columns.
- **Use `JOIN` instead of subqueries** where possible.
- **Limit result sets** with `TOP` or `FETCH`.
- **Use `WHERE` before `JOIN`** to reduce rows early.
- **Avoid functions on columns** in `WHERE` (e.g., `WHERE YEAR(HireDate) = 2023`).
- **Use `EXISTS` instead of `IN`** for large datasets.

---

### **14. SQL Server Security**
#### **Authentication Modes**
- **Windows Authentication**: Integrated security.
- **SQL Authentication**: Username/password.
- **Contained Databases**: Authenticate at database level.

#### **Permissions**
```sql
-- Grant permissions
GRANT SELECT, INSERT ON Employees TO User1;
GRANT EXECUTE ON usp_GetEmployees TO Role1;

-- Revoke permissions
REVOKE SELECT ON Employees FROM User1;

-- Deny permissions (overrides GRANT)
DENY DELETE ON Employees TO User1;

-- Roles
CREATE ROLE DataReaderCustom;
GRANT SELECT ON Employees TO DataReaderCustom;
EXEC sp_addrolemember 'DataReaderCustom', 'User1';
```

#### **Schemas**
```sql
-- Create schema
CREATE SCHEMA HR;
GO
CREATE TABLE HR.Employees (...);

-- Change schema owner
ALTER AUTHORIZATION ON SCHEMA::HR TO NewOwner;
```

#### **Encryption**
```sql
-- Column-level encryption
CREATE COLUMN ENCRYPTION KEY CEK_Auto1
WITH VALUES ('Key1', 'AQAAANCM...');

CREATE COLUMN MASTER KEY CMK_Auto1
WITH KEY_STORE_PROVIDER_NAME = 'MSSQL_CERTIFICATE_STORE';

-- Always Encrypted
ALTER TABLE Employees
ALTER COLUMN SSN VARCHAR(11)
ENCRYPTED WITH (COLUMN_ENCRYPTION_KEY = CEK_Auto1,
               ENCRYPTION_TYPE = DETERMINISTIC);
```

#### **Row-Level Security (RLS)**
```sql
-- Create inline function for RLS
CREATE SCHEMA Security;
GO
CREATE FUNCTION Security.fn_SecurityPredicate(@DepartmentID INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS fn_SecurityPredicate_result
WHERE @DepartmentID = CAST(SESSION_CONTEXT(N'DepartmentID') AS INT);

-- Apply RLS policy
CREATE SECURITY POLICY Security.DepartmentFilter
ADD FILTER PREDICATE Security.fn_SecurityPredicate(DepartmentID) ON dbo.Employees;
```

---

### **15. Backup & Recovery**
```sql
-- Full backup
BACKUP DATABASE AdventureWorks
TO DISK = 'C:\Backups\AdventureWorks.bak'
WITH COMPRESSION, STATS = 10;

-- Differential backup
BACKUP DATABASE AdventureWorks
TO DISK = 'C:\Backups\AdventureWorks_diff.bak'
WITH DIFFERENTIAL, COMPRESSION;

-- Transaction log backup
BACKUP LOG AdventureWorks
TO DISK = 'C:\Backups\AdventureWorks_log.trn'
WITH COMPRESSION;

-- Restore full backup
RESTORE DATABASE AdventureWorks
FROM DISK = 'C:\Backups\AdventureWorks.bak'
WITH REPLACE, STATS = 10;

-- Restore with recovery
RESTORE DATABASE AdventureWorks
FROM DISK = 'C:\Backups\AdventureWorks.bak'
WITH RECOVERY;

-- Point-in-time restore
RESTORE DATABASE AdventureWorks
FROM DISK = 'C:\Backups\AdventureWorks.bak'
WITH NORECOVERY;
RESTORE LOG AdventureWorks
FROM DISK = 'C:\Backups\AdventureWorks_log.trn'
WITH RECOVERY, STOPAT = '2023-10-01T14:00:00';
```

**Recovery Models**:
| Model | Description | Backup Types Supported |
|-------|-------------|------------------------|
| **FULL** | Complete recovery; logs all operations. | Full, Differential, Log |
| **BULK_LOGGED** | Minimal logging for bulk ops. | Full, Differential, Log (limited) |
| **SIMPLE** | No log backups; auto-truncates log. | Full, Differential |

---

### **16. High Availability & Disaster Recovery**
#### **Always On Availability Groups**
```sql
-- Create availability group
CREATE AVAILABILITY GROUP AG1
FOR DATABASE AdventureWorks
REPLICA ON 'SQLServer1' WITH (
  ENDPOINT_URL = 'TCP://SQLServer1:5022',
  AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
  FAILOVER_MODE = AUTOMATIC
),
'SQLServer2' WITH (
  ENDPOINT_URL = 'TCP://SQLServer2:5022',
  AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
  FAILOVER_MODE = AUTOMATIC
);
```

#### **Failover Clustering**
- **Active/Passive**: One node active at a time.
- **Shared Storage**: SAN or iSCSI.

#### **Log Shipping**
- Automated backup/restore to secondary server.

#### **Database Mirroring** (Deprecated in favor of Always On)
```sql
-- Configure mirroring
ALTER DATABASE AdventureWorks
SET PARTNER = 'TCP://SQLServer2:5022';
```

---

### **17. SQL Server Agent**
```sql
-- Create a job
EXEC msdb.dbo.sp_add_job
  @job_name = 'NightlyBackup',
  @enabled = 1,
  @description = 'Full backup of all databases';

-- Add a job step
EXEC msdb.dbo.sp_add_jobstep
  @job_name = 'NightlyBackup',
  @step_name = 'BackupDatabases',
  @subsystem = 'TSQL',
  @command = 'BACKUP DATABASE [AdventureWorks] TO DISK = ''C:\Backups\AdventureWorks.bak'' WITH COMPRESSION;',
  @retry_attempts = 3,
  @retry_interval = 5;

-- Schedule the job
EXEC msdb.dbo.sp_add_jobschedule
  @job_name = 'NightlyBackup',
  @name = 'NightlySchedule',
  @freq_type = 4, -- Daily
  @freq_interval = 1,
  @active_start_time = 010000; -- 1:00 AM

-- Start the job
EXEC msdb.dbo.sp_start_job @job_name = 'NightlyBackup';
```

---

### **18. Common Interview Questions**
#### **Theory**
1. **What is the difference between `WHERE` and `HAVING`?**
   - `WHERE`: Filters rows before grouping.
   - `HAVING`: Filters groups after `GROUP BY`.

2. **Explain ACID properties.**
   - **Atomicity**: All or nothing.
   - **Consistency**: Valid state before/after.
   - **Isolation**: Transactions don’t interfere.
   - **Durability**: Committed changes persist.

3. **What are the types of indexes?**
   - **Clustered**: Physically reorders table (one per table).
   - **Non-clustered**: Logical order (multiple per table).
   - **Unique**: Enforces uniqueness.
   - **Filtered**: Index on a subset of rows.

4. **What is a deadlock? How do you resolve it?**
   - **Deadlock**: Two transactions block each other.
   - **Resolution**:
     - Retry logic in application.
     - Use `NOLOCK` hints (carefully).
     - Optimize transaction order.

5. **Explain the difference between `DELETE`, `TRUNCATE`, and `DROP`.**
   - `DELETE`: Removes rows (logged; can use `WHERE`).
   - `TRUNCATE`: Removes all rows (minimally logged; faster).
   - `DROP`: Removes the table entirely.

#### **Practical**
1. **Write a query to find the second-highest salary.**
   ```sql
   SELECT MAX(Salary) FROM Employees
   WHERE Salary < (SELECT MAX(Salary) FROM Employees);
   -- OR
   SELECT Salary FROM (
     SELECT Salary, DENSE_RANK() OVER (ORDER BY Salary DESC) AS Rank
     FROM Employees
   ) AS RankedSalaries WHERE Rank = 2;
   ```

2. **Find duplicate records in a table.**
   ```sql
   SELECT Name, COUNT(*) AS Count
   FROM Employees
   GROUP BY Name
   HAVING COUNT(*) > 1;
   ```

3. **Write a query to pivot data.**
   ```sql
   SELECT DepartmentID, [2020], [2021], [2022]
   FROM (
     SELECT DepartmentID, Year, Sales
     FROM SalesData
   ) AS SourceTable
   PIVOT (
     SUM(Sales) FOR Year IN ([2020], [2021], [2022])
   ) AS PivotTable;
   ```

4. **Write a recursive CTE to traverse a hierarchy.**
   ```sql
   WITH EmployeeHierarchy AS (
     SELECT ID, Name, ManagerID, 0 AS Level
     FROM Employees
     WHERE ManagerID IS NULL -- Top-level
     UNION ALL
     SELECT e.ID, e.Name, e.ManagerID, eh.Level + 1
     FROM Employees e
     JOIN EmployeeHierarchy eh ON e.ManagerID = eh.ID
   )
   SELECT * FROM EmployeeHierarchy;
   ```

5. **Optimize a slow-running query.**
   - **Steps**:
     1. Check execution plan for bottlenecks.
     2. Add missing indexes.
     3. Rewrite query (e.g., replace subqueries with joins).
     4. Update statistics:
        ```sql
        EXEC sp_updatestats;
        ```

---

### **19. SQL Server 2022 New Features**
- **T-SQL Enhancements**:
  - `STRING_SPLIT` with ordinal positions.
  - `DATE_BUCKET` for time-series aggregation.
- **Performance**: Intelligent Query Processing improvements.
- **Security**: Ledger for tamper-proof data.
- **Azure Synapse Link**: Near real-time analytics.

---
### **20. Quick Reference: T-SQL Functions**
| Category | Functions | Example |
|----------|-----------|---------|
| **String** | `LEN`, `SUBSTRING`, `CHARINDEX`, `REPLACE`, `CONCAT`, `FORMAT` | `SUBSTRING('Hello', 2, 3) -> 'ell'` |
| **Date/Time** | `GETDATE`, `DATEADD`, `DATEDIFF`, `DATEPART`, `FORMAT` | `DATEADD(DAY, 7, GETDATE())` |
| **Math** | `ABS`, `CEILING`, `FLOOR`, `ROUND`, `POWER`, `SQRT` | `ROUND(3.14159, 2) -> 3.14` |
| **Conversion** | `CAST`, `CONVERT`, `TRY_CAST`, `TRY_CONVERT`, `PARSE` | `CONVERT(VARCHAR, GETDATE(), 101) -> 'MM/DD/YYYY'` |
| **Logical** | `IIF`, `CHOSE`, `ISNULL`, `COALESCE`, `NULLIF` | `IIF(Salary > 100000, 'High', 'Low')` |
| **Aggregate** | `SUM`, `AVG`, `COUNT`, `MIN`, `MAX`, `STRING_AGG` | `STRING_AGG(Name, ', ') WITHIN GROUP (ORDER BY Name)` |

---
**Final Tip**: For interviews, focus on **explaining your thought process** (e.g., why you’d use a CTE vs. a temp table) and **performance considerations**. Practice writing queries on paper/whiteboard!

Need a deeper dive into any topic (e.g., query optimization, Always On)?
