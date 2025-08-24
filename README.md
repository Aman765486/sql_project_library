-- Library Management System Project 2.

-- Creating Branch table.
DROP TABLE IF EXISTS branch;
CREATE TABLE branch
	(
		branch_id VARCHAR(10) PRIMARY KEY,
		manager_id VARCHAR(10),
		branch_address VARCHAR(55),
		contact_no VARCHAR(10)
		);

--Creating Employees table

DROP TABLE IF EXISTS employee;
CREATE TABLE employee
	(emp_id VARCHAR(10) PRIMARY KEY,
	emp_name VARCHAR(25),
	position VARCHAR(25),
	salary INT,
	branch_id VARCHAR(25) -- Fk
	);
	
--Creating Book Table

DROP TABLE IF EXISTS book;
CREATE TABLE book
	(isbn VARCHAR(20) PRIMARY KEY,
	book_title VARCHAR(75),
	category VARCHAR(10),
	rental_price FLOAT,
	status VARCHAR(15),
	author VARCHAR(35),
	publisher VARCHAR(55)
	);
ALTER TABLE book
ALTER COLUMN category TYPE  VARCHAR(25);
--Creating Members Table.

DROP TABLE IF EXISTS members;
CREATE TABLE members
	(member_id  VARCHAR(20) PRIMARY KEY,
	member_name  VARCHAR(25),
	member_address  VARCHAR(75),
	reg_date DATE
	);

--Creating Issued_status Table.

DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
	(issued_id VARCHAR(15) PRIMARY KEY,
	issued_member_id VARCHAR(15), -- Fk
	issued_book_name VARCHAR(75),
	issued_date DATE,
	issued_book_isbn VARCHAR(25), -- Fk
	issued_emp_id VARCHAR(15) -- Fk
	);

-- Creating Return_Status Table.

DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
	(return_id VARCHAR(15) PRIMARY KEY,
	issued_id VARCHAR(15),
	return_book_name VARCHAR(75),
	return_date DATE,
	return_book_isbn VARCHAR(25)
	);

-- FOREIGN KEY

ALTER TABLE issued_status
ADD CONSTRAINT fk_members 
FOREIGN KEY (issued_member_id)
REFERENCES members(member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_book
FOREIGN KEY (issued_book_isbn)
REFERENCES book(isbn);

ALTER TABLE issued_status
ADD CONSTRAINT fk_employees
FOREIGN KEY (issued_emp_id)
REFERENCES employee(emp_id);

ALTER TABLE employee
ADD CONSTRAINT fk_branch
FOREIGN KEY (branch_id)
REFERENCES branch(branch_id);

ALTER TABLE return_status
ADD CONSTRAINT fk_issued_status
FOREIGN KEY (issued_id)
REFERENCES issued_status(issued_id);

SELECT * FROM members;



SELECT * FROM branch;


SELECT * FROM employee;

SELECT * FROM return_status;

SELECT * FROM book;

SELECT * FROM issued_status;

-- Project Task

-- Task 1: Create a new record -- "978-1-60129-456-2" , ' To kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')'

INSERT INTO book(isbn, book_title, category, rental_price, status, author, publisher)
VALUES 
('978-1-60129-456-2' , 'To kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

-- Task 2: Update an Existing Member's Address.

UPDATE members 
SET member_address = '125 Main St'
WHERE member_id = 'C101';

-- Task 3: Delete a Records from the Issued status table
-- Objective: Delete the record with issued_id = 'IS107' from the issued_status table.

DELETE FROM issued_status
WHERE  issued_id = 'IS121'

-- Task 4: Retrive All Books Issued by  Specific Employee 
-- Obejective: Select all books issued by the employee with issued_emp_id = 'E101'.

SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';

-- Task 5: List Members Who have Issued More than One Book 
-- Objective: Use Group By to find members who have issued more than one book.

SELECT 
	issued_emp_id,
	COUNT (issued_id) as total_book_issued
FROM issued_status 
GROUP BY issued_emp_id
HAVING COUNT (issued_id) > 1

-- CTAS
-- Task 6: Create Summary Tables: Used CTAS to generate new table based on Query results 
-- Each book and total book_issued_cnt**

CREATE TABLE book_cnts
AS
SELECT 
	b.isbn,
	b.book_title,
	COUNT (ist.issued_id) as no_issued
FROM book as b
JOIN 
issued_status as ist
ON ist.issued_book_isbn = b.isbn
GROUP BY 1, 2;

SELECT * FROM book_cnts

-- Task 7: Retrive All Books in a Specific Category.

SELECT * FROM book 
WHERE category = 'Classic'

-- Task 8: Find Total Rental Income by Category.

SELECT
	b.category,
	SUM (b.rental_price),
	COUNT(*)
FROM book as b
JOIN 
issued_status as ist
ON ist.issued_book_isbn = b.isbn
GROUP BY 1

-- Task 8: List Members who Registered in the last 180 days.

SELECT * FROM members
WHERE reg_date >= CURRENT_DATE -INTERVAL '180 days' 

INSERT INTO members(member_id, member_name, member_address, reg_date)
VALUES
('C120', 'sam', '145 Main St', '2024-04-01'),
('C121', 'jhon', '133 Main St', '2024-06-01');

-- Task 9: List Employees with Their Branch Manager's Name and their branch details.

SELECT 
	e1.*,
	b.manager_id,
	e2.emp_name as manager
FROM employee as e1
JOIN 
branch as b
ON b.branch_id = e1.branch_id 
JOIN 
employee as e2
ON b.manager_id = e2.emp_id

-- Task 10: Create a Table of books with rental price above a certain threshold 7USD.

CREATE TABLE books_price_greater_than_seven
AS
SELECT * FROM book
WHERE rental_price > 7

SELECT * FROM books_price_greater_than_seven

-- Task 11: Retrieve the list of books not yet returned

SELECT 
	DISTINCT ist.issued_book_name 
FROM issued_status as ist
LEFT JOIN 
return_status as rs
ON ist.issued_id = rs.issued_id
WHERE rs.return_id IS NULL

SELECT * FROM return_status

/*
Task 12: Identify Members with overdue books
Write a Query to Identify members who have overdue books (assume a 30-day return period).
Display the member's_id , member's_name, book_title, issued_date, and days overdue.
*/

--issued_status == members == book == return_status
--filter books which is reurn
--overdue > 340

SELECT 
	ist.issued_member_id,
	m.member_name,
	bk.book_title,
	ist.issued_date,
	rs.return_date,
	CURRENT_DATE - ist.issued_date as over_dues_days
FROM issued_status as ist
JOIN 
members as m
	ON m.member_id = ist.issued_member_id
JOIN
book as bk
ON bk.isbn = ist.issued_book_isbn
LEFT JOIN 
return_status as rs
ON rs.issued_id =ist.issued_id
WHERE 
	rs.return_date IS NULL
	AND
	(CURRENT_DATE - ist.issued_date) > 340
ORDER BY 1

/*
Task 13: Update book status on return
Write a query to update the status of book in the books table tp 'yes' when they returned (basedon entries in the 
return_status table).
*/

SELECT * FROM issued_status 
WHERE issued_book_isbn = '978-0-451-52994-2';

SELECT * FROM book
WHERE isbn = '978-0-451-52994-2';

UPDATE book
SET status = 'no'
WHERE isbn = '978-0-451-52994-2';

SELECT * FROM return_status
WHERE issued_id = 'IS130';

--
INSERT INTO return_status(return_id, issued_id, return_date)
VALUES
('RS125', 'IS130', CURRENT_DATE);
SELECT * FROM return_status
WHERE issued_id ='IS130';

--Store Procedures

CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(30), p_return_date DATE)
LANGUAGE plpgsql
AS $$

DECLARE
	v_isbn VARCHAR(50);
	v_book_name VARCHAR(80);
	
BEGIN
	--Inserting into returns based on users input
	INSERT INTO return_status(return_id, issued_id, return_date)
VALUES
('p_return_id', 'p_issued_id', CURRENT_DATE);

	SELECT 
			issued_book_isbn,
			book_title
			INTO
			v_isbn
			v_book_name
		FROM issued_status
		WHERE issued_id = issued_id;
		
	UPDATE book
	SET status = 'yes'
	WHERE isbn = v_isbn;
	
	RAISE NOTICE 'Thank you for returning the book %',v_book_name;
END;
$$

Call add_return_records();

/*
Task 14: Branch Performance Report
Create a query that generates a performance report for each branch, showing the number of book issued, the number of 
books returned, and the total revenue Generated from book rentals
*/

SELECT * FROM branch

SELECT * FROM issued_status;

SELECT * FROM employee;

SELECT * FROM book;

SELECT * FROM return_status

CREATE TABLE branch_reports
AS
SELECT 	
	b.branch_id,
	b.manager_id,
	COUNT (ist.issued_id) as number_book_issued,
	COUNT (rs.return_id) as number_od_book_return,
	SUM(bk.rental_price) as total_revenue
FROM issued_status as ist
JOIN 
employee as e
ON e.emp_id = ist.issued_emp_id
JOIN
branch as b
ON e.branch_id = b.branch_id
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
JOIN 
book as bk
ON ist.issued_book_isbn = bk.isbn
GROUP BY 1, 2;

SELECT * FROM 
/*
Task 15: CTAS: CREATE a table of Active Members
use the create table as (CTAS) statement to create a new table active_members containing members who have isuued at
least one book in the last 12 months.
*/

CREATE TABLE active_members
AS
SELECT * FROM members
WHERE member_id IN(SELECT 
						DISTINCT issued_member_id
					FROM issued_status
					WHERE 
						issued_date >= CURRENT_DATE - 365
					)

SELECT *FROM active_members

/*
Task 17: Find Employees with the most book issued processed
Write a query to find the top 3 employee who have processed the most book issues. Display yhe employess name, numbers
of books processed and their branch.
*/

SELECT 
	e.emp_name,
	b.*,
	COUNT(ist.issued_id) as no_book_isuued
FROM issued_status as ist
JOIN 
employee as e
ON e. emp_id = ist.issued_emp_id
JOIN 
branch as b
ON e.branch_id = b.branch_id
GROUP BY 1, 2


SELECT * FROM active_members;
SELECT * FROM book;
SELECT * FROM book_cnts;
SELECT * FROM book_price_greater_than_seven;
SELECT * FROM branch;
SELECT * FROM branch_reports;
SELECT * FROM employee;
SELECT * FROM issued_status;
SELECT * FROM members;
SELECT * FROM return_status;


--END OF THE PROJECT
