# HadoopHiveHue
Hadoop , Hive, Hue setup pseudo distributed  environment  using docker compose
# Hive Employee and Department Data Analysis

## Project Overview
This project involves analyzing employee and department data using Apache Hive. The tasks include data loading, transformation, partitioning, and performing various queries to extract meaningful insights from the datasets.

## Dataset Description

### **employees.csv**
This dataset contains information about employees, including their department, job role, salary, and project assignment.
- emp_id - Unique employee ID
- name - Employee's full name
- age - Employee's age
- job_role - Designation of the employee
- salary - Annual salary of the employee
- project - Assigned project (One of: Alpha, Beta, Gamma, Delta, Omega)
- join_date - Date when the employee joined
- department - Department to which the employee belongs (Used for partitioning)

### **departments.csv**
This dataset contains information about different departments in the company.
- dept_id - Unique department ID
- department_name - Name of the department
- location - Location of the department

## Hive Implementation Steps

### **1. Load Data into Hive**
```sql
CREATE TABLE temp_employees (
    emp_id INT,
    name STRING,
    age INT,
    job_role STRING,
    salary DOUBLE,
    project STRING,
    join_date STRING,
    department STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

LOAD DATA INPATH '/user/hive/warehouse/employees.csv' INTO TABLE temp_employees;

CREATE TABLE departments (
    dept_id INT,
    department_name STRING,
    location STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

LOAD DATA INPATH '/user/hive/warehouse/departments.csv' INTO TABLE departments;
```

### **2. Transform and Move Data to Partitioned Table**
```sql
CREATE TABLE employees (
    emp_id INT,
    name STRING,
    age INT,
    job_role STRING,
    salary DOUBLE,
    project STRING,
    join_date STRING
)
PARTITIONED BY (department STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

INSERT OVERWRITE TABLE employees PARTITION(department)
SELECT emp_id, name, age, job_role, salary, project, join_date, department FROM temp_employees;
```

### **3. Query Execution**
```sql
-- Retrieve all employees who joined after 2015
SELECT * FROM employees WHERE year(join_date) > 2015;

-- Find the average salary of employees in each department
SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department;

-- Identify employees working on the 'Alpha' project
SELECT * FROM employees WHERE project = 'Alpha';

-- Count the number of employees in each job role
SELECT job_role, COUNT(*) AS num_employees FROM employees GROUP BY job_role;

-- Retrieve employees whose salary is above the average salary of their department
SELECT e.* FROM employees e
JOIN (SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department) avg_table
ON e.department = avg_table.department
WHERE e.salary > avg_table.avg_salary;

-- Find the department with the highest number of employees
SELECT department, COUNT(*) AS num_employees FROM employees
GROUP BY department
ORDER BY num_employees DESC LIMIT 1;

-- Check for employees with null values in any column and exclude them from analysis
SELECT * FROM employees WHERE emp_id IS NOT NULL
AND name IS NOT NULL
AND age IS NOT NULL
AND job_role IS NOT NULL
AND salary IS NOT NULL
AND project IS NOT NULL
AND join_date IS NOT NULL
AND department IS NOT NULL;

-- Join employees and departments tables to display employee details with department locations
SELECT e.*, d.location FROM employees e
JOIN departments d ON e.department = d.department_name;

-- Rank employees within each department based on salary
SELECT emp_id, name, department, salary,
RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees;

-- Find the top 3 highest-paid employees in each department
WITH RankedEmployees AS (
    SELECT emp_id, name, department, salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
    FROM employees
)
SELECT * FROM RankedEmployees WHERE salary_rank <= 3;
```

## **Execution Commands**

### **1. Running the HQL Queries**
```bash
hive -f queries.hql > output.txt
```

### **2. Running Individual Queries**
```bash
hive -e "SELECT * FROM employees WHERE year(join_date) > 2015;" > employees_after_2015.txt
```



## **Expected Output**
- The output of queries will be saved in separate `.txt` files.
- The Hive queries will be in `queries.hql`.
  


