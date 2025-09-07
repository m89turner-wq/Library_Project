# Library Management System using SQL Project

## Project Overview

**Project Title**: Library Management System   
**Database**: `library_project`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![library_Project](https://github.com/m89turner-wq/Library_Project/blob/5f230bb841fbdf37502a792c2a4c412cfe45cbfb/library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilise CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyse and retrieve specific data.

## Project Structure

### 1. Database Setup
![library_Project](https://github.com/m89turner-wq/Library_Project/blob/b19ecee41d9988fa7ba66516eee08a7db7a50c3a/library_schema.png)

- **Database Creation**: Created a database named `library_project`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
-- Library Management System Project

--Create Branch Table

CREATE TABLE branch
	(
		branch_id VARCHAR(10) PRIMARY KEY,
		manager_id VARCHAR(10),
		branch_address VARCHAR(60),
		contact_no VARCHAR(10)
	);


-- Create Employee Table

CREATE TABLE employees
	(
		emp_id VARCHAR(10) PRIMARY KEY,
		emp_name VARCHAR(25),
		position VARCHAR(15),
		salary INT,
		branch_id VARCHAR(25)
	);


-- Create Books Table

CREATE TABLE books
	(
		ISBN VARCHAR(20) PRIMARY KEY,
		book_title VARCHAR(80),
		category VARCHAR(20),
		rental_price FLOAT,
		status VARCHAR(20),
		author VARCHAR(35),
		publisher VARCHAR(25)
	);

ALTER TABLE books
ALTER COLUMN category 

-- Create Members Table

CREATE TABLE members
	(
		member_id VARCHAR(10) PRIMARY KEY,
		member_name VARCHAR(40),
		member_address VARCHAR(80),
		reg_date DATE
	);


-- Create Issue Status Table

CREATE TABLE issued_status
	(
		issued_id VARCHAR(10) PRIMARY KEY,
		issued_member_id VARCHAR(10),
		issued_book_name VARCHAR(80),
		issued_date DATE,
		issued_book_isbn VARCHAR(25),
		issued_emp_id VARCHAR(10)
	);


-- Create return Status Table

CREATE TABLE return_status
	(
		return_id VARCHAR(10) PRIMARY KEY,
		issued_id VARCHAR(10),
		return_book_name VARCHAR(80),
		return_date DATE,
		return_book_isbn VARCHAR(25)
	);


--Foreign Key
ALTER TABLE issued_status
ADD CONSTRAINT fk_members
FOREIGN KEY (issued_member_id)
REFERENCES members(member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_books
FOREIGN KEY (issued_book_isbn)
REFERENCES books(ISBN);

ALTER TABLE issued_status
ADD CONSTRAINT fk_employees
FOREIGN KEY (issued_emp_id)
REFERENCES employees(emp_id);

ALTER TABLE employees
ADD CONSTRAINT fk_branch
FOREIGN KEY (branch_id)
REFERENCES branch(branch_id);

ALTER TABLE return_status
ADD CONSTRAINT fk_issued_status
FOREIGN KEY (issued_id)
REFERENCES issued_status(issued_id);

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1: Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

SELECT * FROM books;
```

**Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member_address = '125 Main St'
WHERE member_id = 'C101';

SELECT * FROM members;
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE   issued_id =   'IS121';

SELECT * FROM issued_status;
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT 
	issued_member_id,
	COUNT(issued_id) as total_books_borrowed
FROM issued_status
GROUP BY issued_member_id
HAVING COUNT (issued_id) > 1
ORDER BY total_books_borrowed DESC;
```

### 3. CTAS (Create Table As Select)

**Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_cnts**

```sql
CREATE TABLE book_cnts
AS
SELECT
	b.isbn,
	b.book_title,
	COUNT(ist.issued_id) AS no_issued
FROM books as b
JOIN
issued_status as ist
ON ist.issued_book_isbn = b.isbn
GROUP BY 1;

SELECT * FROM book_cnts;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

**Task 7: Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Horror';
```

**Task 8: Find Total Rental Income, number unique books and average rental price by Category:**:

```sql
SELECT 
  b.category,
  SUM(bc.no_issued * b.rental_price) AS rental_income,
  COUNT(DISTINCT b.isbn) AS num_unique_books,
  AVG(b.rental_price) AS avg_rental_price
FROM book_cnts bc
JOIN books b ON bc.isbn = b.isbn
GROUP BY 1
ORDER BY rental_income DESC;
```

**Task 9: List Members Who Registered in the Last 180 Days**:
-- no members registered within 180 day in original data so also adding records registered yesterday to test

```sql
INSERT INTO members(member_id, member_name, member_address, reg_date)
VALUES
('C120', 'Michael Turner-Fitzgerald', '101 Street Name', '2025-08-28'),
('C121', ' Jim Halpert', '12 Dunder Mifflin St', '2025-07-31');

SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
```

**Task 10: List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT 
	e1.*,
	b.manager_id,
	e2.emp_name AS manager
FROM employees as e1
JOIN
branch as b
ON b.branch_id = e1.branch_id
JOIN
employees as e2
ON b.manager_id = e2.emp_id;
```

**Task 11. Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
CREATE TABLE premium_price_books AS
SELECT * FROM books
WHERE rental_price > 7;
```

**Task 12: Retrieve the List of Books Not Yet Returned**
```sql
SELECT 
	DISTINCT ist.issued_book_name
FROM issued_status AS ist
LEFT JOIN
return_status AS rs
ON ist.issued_id = rs.issued_id
WHERE rs.return_id IS NULL;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
SELECT 
	ist.issued_member_id,
	m.member_name,
	bk.book_title,
	ist.issued_date,
	CURRENT_DATE - ist.issued_date AS over_due_days
FROM issued_status AS ist
JOIN members AS m
	ON m.member_id = ist.issued_member_id
JOIN books AS bk
	ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status AS rs
	ON rs.issued_id = ist.issued_id 
WHERE 
	rs.return_date IS NULL
	AND 
	(CURRENT_DATE - ist.issued_date) > 30
	ORDER BY 1;
```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

CREATE OR REPLACE PROCEDURE add_return_status(p_return_id VARCHAR(10), p_issued_id VARCHAR(10), p_book_quality VARCHAR(15))
LANGUAGE plpgsql
AS $$

DECLARE 
	v_isbn VARCHAR(20);
	v_book_name VARCHAR (80);
	
BEGIN
	-- Insert into returns from user input
	INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
	VALUES
	(p_return_id, p_issued_id, CURRENT_DATE, p_book_quality);

	SELECT
		issued_book_isbn,
		issued_book_name
		INTO
		v_isbn,
		v_book_name
	FROM issued_status
	WHERE issued_id = p_issued_id;
	
	UPDATE books
	SET status = 'yes'
	WHERE isbn = v_isbn;

	RAISE NOTICE 'Thank you for returning the book: %', v_book_name;

END;
$$

-- Test function 

-- issued_id = IS135
-- isbn 978-0-307-58837-1

SELECT * FROM books
WHERE isbn = '978-0-307-58837-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';

SELECT * FROM return_status
WHERE issued_id = 'IS135';

-- Adding a record. Test confirmed that records added to returns table and status updating in the books table.
CALL add_return_status('RS138', 'IS135', 'Good');

```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
CREATE TABLE branch_reports
AS
SELECT 
	b.branch_id,
	b.manager_id,
	COUNT(ist.issued_id) AS no_of_books_issues,
	COUNT(rs.return_id) AS no_of_books_returned,
	SUM(bk.rental_price) AS total_revenue
FROM issued_status AS ist
JOIN 
employees AS e
ON e.emp_id = ist.issued_emp_id
JOIN 
branch AS b
ON e.branch_id = b.branch_id
LEFT JOIN
return_status AS rs
ON rs.issued_id = ist.issued_id
JOIN
books AS bk
ON ist.issued_book_isbn = bk.isbn
GROUP BY 1, 2;

SELECT * FROM branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 6 months.

```sql

CREATE TABLE active_members
AS
SELECT * FROM members
WHERE member_id IN
	(SELECT
		DISTINCT issued_member_id
		FROM issued_status
		WHERE
			issued_date >= CURRENT_DATE - INTERVAL '6 month'
	);

SELECT * FROM active_members;

```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
SELECT 
	e.emp_name,
	b.*,	
	COUNT(ist.issued_id) AS num_books_processed
FROM issued_status AS ist
JOIN
employees AS e
ON e.emp_id = ist.issued_emp_id
JOIN
branch as b
ON e.branch_id = b.branch_id
GROUP BY 1, 2
ORDER BY num_books_processed DESC
LIMIT 3;
```


**Task 18: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql

CREATE OR REPLACE PROCEDURE issue_book(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), p_issued_book_isbn VARCHAR(30), 
p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
-- Declare variable
v_status VARCHAR(10);

BEGIN
	-- check book avaiable (status = 'yes')
	SELECT 
		status
		INTO
		v_status
	FROM books
	WHERE isbn = p_issued_book_isbn;

	IF v_status = 'yes' THEN
		INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
		VALUES (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);
		
		UPDATE books
		SET status = 'no'
		WHERE isbn = p_issued_book_isbn;

		RAISE NOTICE 'Book record added successfully for book ISBN: %', p_issued_book_isbn;

	ELSE
		RAISE NOTICE 'Sorry, the book you have requested is unavailable. Book ISBN: %', p_issued_book_isbn;

	END IF;

END;
$$

-- test
-- 978-0-553-29698-2 - status = yes
-- 978-0-375-41398-8 - status = no

CALL issue_book('ISS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue_book('ISS156', 'C108', '978-0-375-41398-8', 'E104');

-- both books should show issued = no 
SELECT * FROM books
WHERE isbn IN ('978-0-553-29698-2', '978-0-375-41398-8');

```



**Task 19: Create Table As Select (CTAS)**
Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.

Description: Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 30 days. The table should include:
    The number of overdue books.
    The total fines, with each day's fine calculated at $0.50.
    The number of books issued by each member.
    The resulting table should show:
    Member ID
    Number of overdue books
    Total fines

```sql

CREATE TABLE overdue_summary AS
SELECT 
    ist.issued_member_id AS member_id,
    COUNT(*) AS number_of_overdue_books,
    SUM((CURRENT_DATE - ist.issued_date) * 0.50) AS total_fines
FROM issued_status AS ist
LEFT JOIN return_status AS rs
    ON rs.issued_id = ist.issued_id
WHERE 
    rs.return_date IS NULL
    AND (CURRENT_DATE - ist.issued_date) > 30
GROUP BY ist.issued_member_id;

SELECT * FROM overdue_summary;
```

## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

## Power BI Dashboard

> 📌 *This dashboard was built to test importing structured data from PostgreSQL into Power BI and explore interactive reporting. Despite the limited dataset, it demonstrates core integration and visualisation capabilities.*

https://app.powerbi.com/view?r=eyJrIjoiODI1MTA4ZDktOTAxZi00ZjY0LWE4MTItZWEyNDJkODkxZmM2IiwidCI6ImRiMjk3ODAyLTZjZTAtNDIyOC05MDU3LWM5NDhlY2I0YTc5NyJ9

## 📊 Power BI Dashboard: Library Management System

As part of my data visualisation and integration portfolio, I developed a **Library Management Dashboard** using **Power BI**, connected directly to a relational dataset hosted in **PostgreSQL**. This project demonstrates the full pipeline from SQL-based modelling to interactive dashboard design.

### 🔗 Data Integration

I imported multiple normalised tables from PostgreSQL, including:
- `books`, `members`, `issued_status`, `return_status`, `employees`, `branch_reports`, `overdue_summary`
- A custom `Calendar` table to support time intelligence

Relationships were defined to reflect the schema’s logic, including indirect paths such as `issued_status → employees → branch`, enabling dynamic filtering across visuals.

### 🧠 Dashboard Features

The dashboard includes:

- **Date and Branch Slicers**: Powered by the `Calendar` table and relational links
- **KPI Cards**:
  - Total Books Issued
  - Total Fines (based on overdue logic at $0.50/day)
  - New Members Registered in the Last 6 Months
- **Column Charts**:
  - Total Revenue by Branch
  - Total Books Issued by Branch
  - Total Books Issued by Employee
- **Matrix Heatmap**:
  - Books Issued by Day of Week
- **Donut Chart**:
  - Available vs Unavailable Books
- **Members Table**:
  - Count of Overdue Books per Member

### 🧪 Purpose & Learnings

While the dataset was intentionally limited in scope, the project served as a successful proof-of-concept for:
- Connecting PostgreSQL to Power BI using native connectors
- Modelling complex relationships across multiple tables
- Creating responsive visuals using DAX and slicers
- Designing a clean, stakeholder-friendly layout


## Author - Michael Turner-Fitzgerald

Hi, I'm Michael Turner-Fitzgerald, a financial services professional pivoting into data analytics and visualisation. With a background in risk management and relationship strategy, I bring a unique blend of precision, creativity, and business acumen to every project. This SQL analysis is part of a broader portfolio that includes interactive dashboards, analysis, and data storytelling.

- **Website**: https://www.mturnfitz.co.uk/
- **LinkedIn**: https://www.linkedin.com/in/michael-turner-fitzgerald-624953197/

Thanks for exploring this project. I welcome feedback, collaboration, and conversation.
