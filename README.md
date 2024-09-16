# Problems-on-Recursive-CTE

## Table:

```sql
-- Create the Employees table
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    EmployeeName VARCHAR(100),
    ManagerID INT,
    Salary DECIMAL(10, 2),
    FOREIGN KEY (ManagerID) REFERENCES Employees(EmployeeID)
);

-- Insert sample data into Employees
INSERT INTO Employees (EmployeeID, EmployeeName, ManagerID, Salary) VALUES
(1, 'Alice', NULL, 100000),
(2, 'Bob', 1, 90000),
(3, 'Charlie', 1, 80000),
(4, 'David', 2, 75000),
(5, 'Eve', 2, 70000),
(6, 'Frank', 3, 72000),
(7, 'Grace', 3, 68000);
```

## Hierarchy Retrieval
**Question:** Write a query to retrieve the hierarchy of employees, displaying the EmployeeName, their ManagerName, and the level in the hierarchy (e.g., Alice is at level 1, Bob and Charlie are at level 2, etc.).


```sql
WITH RECURSIVE EmployeeHierarchy AS (
    -- Base case: Employees with no manager (top-level employees)
    SELECT 
        EmployeeID,
        EmployeeName,
        ManagerID,
        Salary,
        1 AS Level -- Top level
    FROM Employees
    WHERE ManagerID IS NULL
    UNION ALL
    -- Recursive case: Employees who have a manager
    SELECT 
        e.EmployeeID,
        e.EmployeeName,
        e.ManagerID,
        e.Salary,
        eh.Level + 1 AS Level
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
-- Final select to get the manager's name and level
SELECT 
    eh.EmployeeName,
    m.EmployeeName AS ManagerName,
    eh.Level
FROM EmployeeHierarchy eh
LEFT JOIN Employees m ON eh.ManagerID = m.EmployeeID
ORDER BY eh.Level, eh.EmployeeName;
```


## Total Salary under Each Manager
**Question:** Write a query to calculate the total salary for each employee, including the sum of salaries for all employees under their management at any level. The result should include EmployeeID, EmployeeName, ManagerID, ManagerName, and the TotalSalary under their management.

```sql
WITH RECURSIVE EmployeeHierarchy AS (
    -- Base case: Select the direct employees
    SELECT 
        e.EmployeeID, 
        e.EmployeeName, 
        e.ManagerID, 
        e.Salary,
        e.EmployeeID AS RootEmployeeID
    FROM Employees e
    UNION ALL
    -- Recursive case: Select employees under each manager
    SELECT 
        e.EmployeeID, 
        e.EmployeeName, 
        e.ManagerID, 
        e.Salary,
        eh.RootEmployeeID
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
-- Final query to aggregate the total salary
SELECT 
    eh.RootEmployeeID AS EmployeeID,
    e.EmployeeName,
    e.ManagerID,
    m.EmployeeName AS ManagerName,
    SUM(eh.Salary) AS TotalSalary
FROM EmployeeHierarchy eh
INNER JOIN Employees e ON eh.RootEmployeeID = e.EmployeeID
LEFT JOIN Employees m ON e.ManagerID = m.EmployeeID
GROUP BY eh.RootEmployeeID, e.EmployeeName, e.ManagerID, m.EmployeeName
ORDER BY eh.RootEmployeeID;
```


## Highest Salary Employee at Each Level
**Question:** Write a query to find the employee with the highest salary at each level of the hierarchy.

```sql
WITH RECURSIVE EmployeeHierarchy AS (
    -- Anchor member: Start with the top-level employees (those with no manager)
    SELECT 
        EmployeeID,
        EmployeeName,
        ManagerID,
        Salary,
        0 AS Level
    FROM Employees
    WHERE ManagerID IS NULL
    UNION ALL
    -- Recursive member: Find employees at the next level in the hierarchy
    SELECT 
        e.EmployeeID,
        e.EmployeeName,
        e.ManagerID,
        e.Salary,
        eh.Level + 1 AS Level
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
),
-- Rank employees by salary within each level
RankedEmployees AS (
    SELECT 
        EmployeeID,
        EmployeeName,
        ManagerID,
        Salary,
        Level,
        DENSE_RANK() OVER (PARTITION BY Level ORDER BY Salary DESC) AS D_rnk
    FROM EmployeeHierarchy
)
-- Select the employees with rank 1 (highest salary) at each level
SELECT 
    EmployeeID,
    EmployeeName,
    ManagerID,
    Salary,
    Level
FROM RankedEmployees
WHERE D_rnk = 1
ORDER BY Level, EmployeeID;
```

## By Rithul Shaji
