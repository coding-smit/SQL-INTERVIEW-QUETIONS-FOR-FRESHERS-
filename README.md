# SQL Interview Revision Sheet

A quick-revision guide covering **Round 1** and **Round 2** of SQL interview practice. Each question includes the problem, the correct query, a step-by-step explanation, and before/after tables.

![SQL](https://img.shields.io/badge/SQL-Interview%20Prep-blue)
![Questions](https://img.shields.io/badge/Questions-20-green)
![Level](https://img.shields.io/badge/Level-Beginner%20to%20Intermediate-orange)

---

## Table of Contents

- [Topics Covered](#topics-covered)
- [Round 1](#round-1)
  - [Q1. DELETE vs TRUNCATE vs DROP](#q1-delete-vs-truncate-vs-drop)
  - [Q2. Employee with the Highest Salary](#q2-employee-with-the-highest-salary)
  - [Q3. WHERE vs HAVING](#q3-where-vs-having)
  - [Q4. PRIMARY KEY vs UNIQUE](#q4-primary-key-vs-unique)
  - [Q5. Average Salary by Department](#q5-average-salary-by-department)
  - [Q6. JOIN Two Tables](#q6-join-two-tables)
  - [Q7. INNER vs LEFT vs RIGHT JOIN](#q7-inner-vs-left-vs-right-join)
  - [Q8. Second Highest Salary](#q8-second-highest-salary)
  - [Q9. Departments with More Than 2 Employees](#q9-departments-with-more-than-2-employees)
  - [Q10. Count Employees Under Each Manager](#q10-count-employees-under-each-manager)
- [Round 2](#round-2)
  - [Q1. Salary Greater Than Overall Average](#q1-salary-greater-than-overall-average)
  - [Q2. Highest Salary in Each Department](#q2-highest-salary-in-each-department)
  - [Q3. Third Highest Distinct Salary](#q3-third-highest-distinct-salary)
  - [Q4. Departments Where Average Salary > 65000](#q4-departments-where-average-salary--65000)
  - [Q5. Find Duplicate Salaries](#q5-find-duplicate-salaries)
  - [Q6. Highest-Paid Employee](#q6-highest-paid-employee)
  - [Q7. Second Highest Salary in Each Department](#q7-second-highest-salary-in-each-department)
  - [Q8. SELF JOIN: Employee and Manager](#q8-self-join-employee-and-manager)
  - [Q9. Highest-Paid per Department (Window Function)](#q9-highest-paid-per-department-window-function)
  - [Q10. Salary Greater Than Department Average](#q10-salary-greater-than-department-average)
- [Key Interview Lessons](#key-interview-lessons)

---

## Topics Covered

```text
SQL
│
├── Basic Commands
│   ├── DELETE
│   ├── TRUNCATE
│   └── DROP
│
├── Aggregation
│   ├── COUNT()
│   ├── MAX()
│   ├── AVG()
│   └── GROUP BY
│
├── Filtering
│   ├── WHERE
│   └── HAVING
│
├── Keys
│   ├── PRIMARY KEY
│   └── UNIQUE
│
├── JOINs
│   ├── INNER JOIN
│   ├── LEFT JOIN
│   ├── RIGHT JOIN
│   └── SELF JOIN
│
├── Subqueries
│   ├── Maximum salary
│   ├── Second highest
│   ├── Third highest
│   └── Average comparison
│
└── Window Functions
    ├── ROW_NUMBER()
    └── DENSE_RANK()
```

---

# Round 1

## Q1. DELETE vs TRUNCATE vs DROP

**Question:** What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?

| Command    | What it removes   | Table structure | `WHERE` | Rollback\*   |
| ---------- | ----------------- | --------------- | :-----: | ------------ |
| `DELETE`   | Selected/all rows | Remains         |   Yes   | Usually yes  |
| `TRUNCATE` | All rows          | Remains         |    No   | DB-dependent |
| `DROP`     | Entire table      | Removed         |    No   | DB-dependent |

**Before**

```text
employee
----------------
id | name
1  | A
2  | B
3  | C
```

**DELETE**

```sql
DELETE FROM employee
WHERE id = 2;
```

```text
id | name
1  | A
3  | C
```

**TRUNCATE**

```sql
TRUNCATE TABLE employee;
```

```text
employee
----------------
(empty)
```

The table still exists.

**DROP**

```sql
DROP TABLE employee;
```

```text
employee
----------------
Table does not exist
```

> **Interview tip:** Don't simply say "DELETE can always be recovered." Rollback behavior depends on the database and transaction handling.

[Back to top](#sql-interview-revision-sheet)

---

## Q2. Employee with the Highest Salary

**Question:** Find the employee with the highest salary, including the complete row.

```text
employee
-----------------------------
id | name | salary
1  | A    | 30000
2  | B    | 50000
3  | C    | 70000
4  | D    | 60000
```

**Common mistake**

```sql
SELECT MAX(salary), *
FROM employee;
```

> `MAX()` is an aggregate, while `*` contains individual columns. They can't be mixed this way.

**Correct answer**

```sql
SELECT *
FROM employee
WHERE salary = (
    SELECT MAX(salary)
    FROM employee
);
```

**How it works**

The inner query returns the maximum salary:

```sql
SELECT MAX(salary) FROM employee;  -- 70000
```

The outer query then finds the row where `salary = 70000`.

**Result**

```text
id | name | salary
3  | C    | 70000
```

[Back to top](#sql-interview-revision-sheet)

---

## Q3. WHERE vs HAVING

- **`WHERE`** filters **rows** (before grouping).
- **`HAVING`** filters **groups** (after grouping).

**WHERE example**

```sql
SELECT *
FROM employee
WHERE salary > 50000;
```

| Before (name, dept, salary) | After (salary > 50000) |
| --------------------------- | ---------------------- |
| A, IT, 30000                |                        |
| B, IT, 70000                | B, IT, 70000           |
| C, HR, 60000                | C, HR, 60000           |

**HAVING example**

```sql
SELECT department, COUNT(*) AS employee_count
FROM employee
GROUP BY department
HAVING COUNT(*) > 2;
```

**Easy memory**

```text
WHERE  → Rows
HAVING → Groups
```

[Back to top](#sql-interview-revision-sheet)

---

## Q4. PRIMARY KEY vs UNIQUE

| Feature                       | PRIMARY KEY | UNIQUE                     |
| ----------------------------- | :---------: | -------------------------- |
| Uniquely identifies each row  |     Yes     | Yes                        |
| Allows `NULL`                 |      No     | Depends on the database    |
| Allows duplicates             |      No     | No                         |
| How many allowed per table    |     One     | Multiple                   |

**Example**

```sql
CREATE TABLE employee (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    name VARCHAR(50)
);
```

| id | name | email       |
| -: | ---- | ----------- |
|  1 | A    | a@gmail.com |
|  2 | B    | b@gmail.com |

```text
id    → PRIMARY KEY
email → UNIQUE
```

[Back to top](#sql-interview-revision-sheet)

---

## Q5. Average Salary by Department

```text
employee
-----------------------------
id | name | department | salary
1  | A    | IT         | 30000
2  | B    | HR         | 40000
3  | C    | IT         | 50000
4  | D    | HR         | 60000
5  | E    | IT         | 70000
```

**Answer**

```sql
SELECT department, AVG(salary) AS average_salary
FROM employee
GROUP BY department;
```

**Result**

```text
department | average_salary
-----------|---------------
IT         | 50000
HR         | 50000
```

```text
IT = (30000 + 50000 + 70000) / 3 = 50000
HR = (40000 + 60000) / 2         = 50000
```

[Back to top](#sql-interview-revision-sheet)

---

## Q6. JOIN Two Tables

**employee**

```text
id | name | dept_id
1  | A    | 10
2  | B    | 20
3  | C    | 10
```

**department**

```text
dept_id | dept_name
10      | IT
20      | HR
30      | Sales
```

**Answer**

```sql
SELECT e.name, d.dept_name
FROM employee e
JOIN department d
  ON e.dept_id = d.dept_id;
```

**Result**

```text
name | dept_name
-----|----------
A    | IT
B    | HR
C    | IT
```

[Back to top](#sql-interview-revision-sheet)

---

## Q7. INNER vs LEFT vs RIGHT JOIN

**INNER JOIN**: only matching rows.

```text
LEFT TABLE       RIGHT TABLE
A ────────────── A
B ────────────── B
C                D
```

Result: `A`, `B`

**LEFT JOIN**: all rows from the left table + matching right rows.

```text
A → A
B → B
C → NULL
```

**RIGHT JOIN**: all rows from the right table + matching left rows.

```text
A    → A
B    → B
NULL → D
```

**Easy memory**

```text
INNER → Matching only
LEFT  → All LEFT
RIGHT → All RIGHT
```

> Remember: **INNER JOIN is not the same thing as SELF JOIN.**

[Back to top](#sql-interview-revision-sheet)

---

## Q8. Second Highest Salary

**Before**

```text
salary
------
30000
50000
50000
70000
90000
```

**Answer**

```sql
SELECT MAX(salary)
FROM employee
WHERE salary < (
    SELECT MAX(salary)
    FROM employee
);
```

**Process**

1. Highest salary = `90000`.
2. Keep only salaries below `90000`.
3. Take the maximum of the rest.

**Result**

```text
70000
```

[Back to top](#sql-interview-revision-sheet)

---

## Q9. Departments with More Than 2 Employees

**Before**

```text
department
-----------
IT
IT
IT
HR
HR
Sales
```

**Answer**

```sql
SELECT department, COUNT(*) AS employee_count
FROM employee
GROUP BY department
HAVING COUNT(*) > 2;
```

**Result**

```text
department | employee_count
-----------|---------------
IT         | 3
```

[Back to top](#sql-interview-revision-sheet)

---

## Q10. Count Employees Under Each Manager

**Before**

```text
id | name | manager_id
1  | A    | NULL
2  | B    | 1
3  | C    | 1
4  | D    | 2
5  | E    | 2
6  | F    | 3
```

**Answer**

```sql
SELECT manager_id, COUNT(*) AS employee_count
FROM employee
WHERE manager_id IS NOT NULL
GROUP BY manager_id;
```

**Result**

```text
manager_id | employee_count
-----------|---------------
1          | 2
2          | 2
3          | 1
```

**Why `WHERE` instead of `HAVING`?**
`manager_id IS NOT NULL` filters individual rows, so `WHERE` is the appropriate clause.

[Back to top](#sql-interview-revision-sheet)

---

# Round 2

## Q1. Salary Greater Than Overall Average

**Before**

```text
name | salary
-----|-------
A    | 50000
B    | 70000
C    | 60000
D    | 80000
E    | 90000
```

Average = `70000`

**Answer**

```sql
SELECT name, salary
FROM employee
WHERE salary > (
    SELECT AVG(salary)
    FROM employee
);
```

**Result**

```text
name | salary
-----|-------
D    | 80000
E    | 90000
```

[Back to top](#sql-interview-revision-sheet)

---

## Q2. Highest Salary in Each Department

**Question:** Find the employee(s) with the highest salary in each department.

**Important difference**

```sql
SELECT department, MAX(salary)
FROM employee
GROUP BY department;
```

This finds the **maximum salary**, but it doesn't properly return **which employee** has that salary.

**Correct answer (correlated subquery)**

```sql
SELECT e.name, e.department, e.salary
FROM employee e
WHERE e.salary = (
    SELECT MAX(e2.salary)
    FROM employee e2
    WHERE e2.department = e.department
);
```

**Before**

```text
name | department | salary
-----|------------|-------
A    | IT         | 50000
B    | IT         | 70000
C    | IT         | 70000
D    | HR         | 60000
E    | HR         | 80000
```

**Result**

```text
name | department | salary
-----|------------|-------
B    | IT         | 70000
C    | IT         | 70000
E    | HR         | 80000
```

Both **B and C** appear because both have the highest IT salary.

[Back to top](#sql-interview-revision-sheet)

---

## Q3. Third Highest Distinct Salary

**Before**

```text
salary
------
50000
70000
70000
80000
90000
```

Distinct salaries ranked:

```text
90000 → 1st
80000 → 2nd
70000 → 3rd
50000 → 4th
```

**Answer**

```sql
SELECT MAX(salary)
FROM employee
WHERE salary < (
    SELECT MAX(salary)
    FROM employee
    WHERE salary < (
        SELECT MAX(salary)
        FROM employee
    )
);
```

**Result**

```text
70000
```

> Select only the salary column. Including other columns such as `employee_name` is unnecessary and breaks the aggregate.

[Back to top](#sql-interview-revision-sheet)

---

## Q4. Departments Where Average Salary > 65000

**Answer**

```sql
SELECT department
FROM employee
GROUP BY department
HAVING AVG(salary) > 65000;
```

**Before (department averages)**

```text
department | average
-----------|--------
IT         | 60000
HR         | 70000
Sales      | 90000
```

**Result**

```text
department
----------
HR
Sales
```

`HAVING` is required because we are filtering on an aggregate result.

[Back to top](#sql-interview-revision-sheet)

---

## Q5. Find Duplicate Salaries

**Before**

```text
salary
------
50000
70000
70000
80000
90000
```

**Answer**

```sql
SELECT salary, COUNT(*) AS salary_count
FROM employee
GROUP BY salary
HAVING COUNT(*) > 1;
```

**Result**

```text
salary | salary_count
-------|-------------
70000  | 2
```

[Back to top](#sql-interview-revision-sheet)

---

## Q6. Highest-Paid Employee

**Question:** Find the employee with the highest salary and return name, department and salary.

**Answer**

```sql
SELECT name, department, salary
FROM employee
WHERE salary = (
    SELECT MAX(salary)
    FROM employee
);
```

**Before**

```text
name | department | salary
-----|------------|-------
A    | IT         | 50000
B    | IT         | 70000
C    | HR         | 60000
D    | HR         | 80000
E    | Sales      | 90000
```

**Result**

```text
name | department | salary
-----|------------|-------
E    | Sales      | 90000
```

[Back to top](#sql-interview-revision-sheet)

---

## Q7. Second Highest Salary in Each Department

This is where **window functions** become useful.

**Answer**

```sql
SELECT name, department, salary
FROM (
    SELECT name,
           department,
           salary,
           ROW_NUMBER() OVER (
               PARTITION BY department
               ORDER BY salary DESC
           ) AS rn
    FROM employee
) AS x
WHERE rn = 2;
```

**How `ROW_NUMBER()` works**

IT:

```text
name | salary | rn
-----|--------|---
B    | 70000  | 1
C    | 70000  | 2
A    | 50000  | 3
```

HR:

```text
name | salary | rn
-----|--------|---
E    | 80000  | 1
D    | 60000  | 2
```

> **Watch out:** `ROW_NUMBER()` gives tied salaries different numbers (see B and C above). If you want the **second-highest DISTINCT salary**, use `DENSE_RANK()` instead.

[Back to top](#sql-interview-revision-sheet)

---

## Q8. SELF JOIN: Employee and Manager

**Before**

```text
id | name | manager_id
---|------|-----------
1  | A    | NULL
2  | B    | 1
3  | C    | 1
4  | D    | 2
5  | E    | 2
```

**Answer**

```sql
SELECT f.name AS employee_name,
       e.name AS manager_name
FROM employee e
JOIN employee f
  ON e.id = f.manager_id;
```

**Result**

```text
employee_name | manager_name
--------------|-------------
B             | A
C             | A
D             | B
E             | B
```

The same table is used twice:

```text
e → manager
f → employee
```

This is called a **SELF JOIN**.

[Back to top](#sql-interview-revision-sheet)

---

## Q9. Highest-Paid per Department (Window Function)

**Answer**

```sql
SELECT name, department, salary
FROM (
    SELECT name,
           department,
           salary,
           DENSE_RANK() OVER (
               PARTITION BY department
               ORDER BY salary DESC
           ) AS rn
    FROM employee
) AS x
WHERE rn = 1;
```

**Before**

```text
name | department | salary
-----|------------|-------
A    | IT         | 50000
B    | IT         | 70000
C    | IT         | 70000
D    | HR         | 60000
E    | HR         | 80000
F    | Sales      | 90000
```

**Ranking**

```text
IT:     B → 1, C → 1, A → 2
HR:     E → 1, D → 2
Sales:  F → 1
```

**Result**

```text
name | department | salary
-----|------------|-------
B    | IT         | 70000
C    | IT         | 70000
E    | HR         | 80000
F    | Sales      | 90000
```

`DENSE_RANK()` returns **both B and C** because they tie for rank 1.

[Back to top](#sql-interview-revision-sheet)

---

## Q10. Salary Greater Than Department Average

**Answer**

```sql
SELECT e.name, e.department
FROM (
    SELECT department,
           AVG(salary) AS average_salary
    FROM employee
    GROUP BY department
) AS x
JOIN employee e
  ON x.department = e.department
WHERE e.salary > x.average_salary;
```

**Before**

```text
name | department | salary
-----|------------|-------
A    | IT         | 50000
B    | IT         | 70000
C    | IT         | 70000
D    | HR         | 60000
E    | HR         | 80000
F    | Sales      | 90000
```

**Department averages**

```text
IT    → 63333.33
HR    → 70000
Sales → 90000
```

**Compare each employee**

```text
A → 50000 < 63333  ❌
B → 70000 > 63333  ✅
C → 70000 > 63333  ✅

D → 60000 < 70000  ❌
E → 80000 > 70000  ✅

F → 90000 = 90000  ❌
```

**Result**

```text
name | department
-----|-----------
B    | IT
C    | IT
E    | HR
```

[Back to top](#sql-interview-revision-sheet)

---

# Key Interview Lessons

**1. `MAX()` does not automatically give you the employee row.**
`MAX(salary)` returns a **value**, not the corresponding employee. Use a subquery or window function to get the full row.

**2. `WHERE` filters rows; `HAVING` filters groups.**

**3. For "Nth highest salary":**

- Simple cases: `MAX` + nested subquery.
- Advanced cases: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`.

**4. For "highest salary per department":**
`GROUP BY` can find the salary, while a **subquery or window function** identifies the employee.

**5. `SELF JOIN` means joining a table with itself**, commonly used for employee-manager relationships.

### Ranking functions at a glance

| Function         | Ties                          | Gaps after ties |
| ---------------- | ----------------------------- | :-------------: |
| `ROW_NUMBER()`   | Different numbers             |       n/a       |
| `RANK()`         | Same rank                     |       Yes       |
| `DENSE_RANK()`   | Same rank                     |        No       |

---

*Happy practicing, and good luck with your interview!*
