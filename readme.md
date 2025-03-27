-- Library Management System using SQL Project --P2
-- Project Overview
-- Project Title: Library Management System
-- Database: library_db

-- This project demonstrates the implementation of a Library Management System using SQL. 
-- It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. 
-- The goal is to showcase skills in database design, manipulation, and querying.

-- Objectives
-- 1.Set up the Library Management System Database: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
-- 2.CRUD Operations: Perform Create, Read, Update, and Delete operations on the data.
-- 3.CTAS (Create Table As Select): Utilize CTAS to create new tables based on query results.
-- 4.Advanced SQL Queries: Develop complex queries to analyze and retrieve specific data.

-- Project Structure
-- 1. Database Setup

![ERD](https://github.com/Nidhi9608/library_db/blob/main/EER_library.png)

create table images (
    id int auto_increment primary key,
    image_name varchar(255),
    image_data longblob
);

insert into images (image_name, image_data) 
values ('EER_library.png', load_file("R:\Git\Library-System-Management---P2\upload github\EER_library.png"));

select 'EER_library.png', "R:\Git\Library-System-Management---P2\upload github\EER_library.png" FROM images;

-- Creating branch table
create table branch
(
branch_id varchar(10) primary key,	
manager_id	varchar(10),
branch_address varchar(55),
contact_no varchar(10)
);
alter table branch
modify column contact_no varchar(20);
select * from branch;

-- Creating table employees
create table employees
(
emp_id varchar(10) primary key,	
emp_name varchar(25),	
position varchar(15),	
salary int,
branch_id varchar(25)
);
select * from employees;

-- Create table books
create table books
(
isbn varchar(20) primary key,
book_title varchar(75),
category varchar(10),
rental_price float,	
status varchar(15),
author varchar(35),	
publisher varchar(55)
);
alter table books
modify column category varchar(25);
select * from books;

-- Creating table members
create table members
(
member_id varchar(10) primary key,
member_name	varchar(25),
member_address varchar(75),
reg_date date
);
select * from members;

-- Creating table issued_status
create table issued_status
(
issued_id  varchar(10) primary key,
issued_member_id varchar(10),	
issued_book_name varchar(75),
issued_date	date,
issued_book_isbn varchar(25),
issued_emp_id varchar(10)
);
select * from issued_status;

-- Creating table return_status
create table return_status
(
return_id varchar(10) primary key,
issued_id varchar(10),
return_book_name varchar(75),	
return_date	date,
return_book_isbn varchar(20)
);
select * from return_status;

-- Foreign Key
alter table issued_status
add constraint fk_members
foreign key (issued_member_id)
references members(member_id);

alter table issued_status
add constraint fk_books
foreign key (issued_book_isbn)
references books(isbn);

alter table issued_status
add constraint fk_employees
foreign key (issued_emp_id)
references employees(emp_id);

alter table return_status
add constraint fk_issued_statu
foreign key (issued_id)
references issued_status(issued_id);

alter table employees
add constraint fk_branch
foreign key (branch_id)
references branch(branch_id);

select * from books;
select * from branch;
select * from employees;
select * from issued_status;
select * from members;
select * from return_status;

-- Project Task

-- CRUD Operations

-- Task 1. Create a New Book Record 
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"
insert into books (isbn, book_title, category, rental_price, status, author, publisher)
values ('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
select * from books;

-- Task 2: Update an Existing Member's Address
update members
set member_address = '125 Main St'
where member_id = 'C101';
select * from members;

-- Task 3: Delete a Record from the Issued Status Table 
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.
select * from issued_status;
delete from issued_status
where issued_id = 'IS121';
select * from issued_status;

-- Task 4: Retrieve All Books Issued by a Specific Employee 
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
select * from issued_status
where issued_emp_id = 'E101';

-- Task 5: List Members Who Have Issued More Than One Book 
-- Objective: Use GROUP BY to find members who have issued more than one book.
select * from issued_status;
select 
issued_member_id,
count(*) as total_books_issued
from issued_status
group by 1
having count(*) > 1 ;

-- CTAS (Create Table As Select)
-- Task 6: Create Summary Tables: Used CTAS to generate new tables based on query results 
-- each book and total book_issued_cnt**
create table book_count as
select 
b.isbn,
book_title,
count(ist.issued_id) as no_issued 
from books as b
join issued_status as ist
on ist.issued_book_isbn = b.isbn
group by 1, 2;

-- 4. Data Analysis & Findings
-- The following SQL queries were used to address specific questions:

-- Task 7. Retrieve All Books in a Specific Category : 'Classic'
select * from books
where category = 'Classic';

-- Task 8: Find Total Rental Income by Category:
select * from issued_status;
select 
b.category,
sum(b.rental_price),
count(*)
from 
issued_status as ist
join
books as b
on b.isbn = ist.issued_book_isbn
group by 1;

-- Task 9: List Members Who Registered in the Last 180 Days:
select * from members
where reg_date >= curdate() - interval 180 day; 

-- Task 10: List Employees with Their Branch Manager's Name and their branch details:
select 
e1.emp_id,
e1.emp_name,
e1.position,
e1.salary,
b.*,
e2.emp_name as manager 
from employees as e1
join branch as b
on e1.branch_id = b.branch_id
join
employees as e2
on e2.emp_id = b.manager_id;

-- Task 11: Create a Table of Books with Rental Price Above a Certain Threshold:
-- rental price above 6
create table expensive_books as
select 
book_title
from books
where rental_price > 6;

-- Task 12:  Retrieve the List of Books Not Yet Returned:
select 
ist.issued_book_name
from issued_status as ist
left join return_status as rst
on rst.issued_id = ist.issued_id
where rst.issued_id is null;

-- Task 13: Identify Members with Overdue Books:
-- Write a query to identify members who have overdue books (assume a 30-day return period). 
-- Display the member's_id, member's name, book title, issue date, and days overdue.
select 
ists.issued_member_id, 
m.member_name, 
bk.book_title, 
ists.issued_date,
current_date() - ists.issued_date as overdue_days
from issued_status as ists
join members as m
on m.member_id = ists.issued_member_id
join books as bk
on bk.isbn = ists.issued_book_isbn
left join return_status as rs
on rs.issued_id = ists.issued_id
where rs.return_date is null
and (current_date() - ists.issued_date) > 30
order by 1
;
 
 
 -- Adding new column in return_status
ALTER TABLE return_status
ADD Column book_quality VARCHAR(15) DEFAULT('Good');

UPDATE return_status
SET book_quality = 'Damaged'
WHERE issued_id 
    IN ('IS112', 'IS117', 'IS118');
SELECT * FROM return_status;

-- Task 14: Update Book Status on Return
-- Write a query to update the status of books in the books table to "Yes" 
-- when they are returned (based on entries in the return_status table).
select * from return_status;
insert into return_status(return_id, issued_id, return_date, book_quality)
values ('RS125' , 'IS130', current_date, 'Good'); 

-- Store Procedures
DELIMITER //

CREATE PROCEDURE add_return_records(
    IN p_return_id VARCHAR(10), 
    IN p_issued_id VARCHAR(10), 
    IN p_book_quality VARCHAR(10)
)
BEGIN
    DECLARE v_isbn VARCHAR(50);
    DECLARE v_book_name VARCHAR(80);
    
    -- Insert into return_status table
    INSERT INTO return_status (return_id, issued_id, return_date, book_quality)
    VALUES (p_return_id, p_issued_id, CURDATE(), p_book_quality);

    -- Fetch book details from issued_status
    SELECT issued_book_isbn, issued_book_name
    INTO v_isbn, v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    -- Update book status
    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    SELECT CONCAT('Thank you for returning the book: ', v_book_name) AS Message;

END //

DELIMITER ;

call add_return_records();

-- Testing functions add_return_records()
-- issued_id = IS135
-- isbn = where isbn = '978-0-307-58837-1'

select * from books
where isbn = '978-0-307-58837-1';

select * from issued_status
where issued_id = 'IS135';

select * from return_status
where issued_id = 'IS135';

select * from issued_status
where issued_book_isbn = '978-0-307-58837-1';

call add_return_records('RS138', 'IS135', 'Good');

-- Delete from return_status the record of issued_id
-- Then the status of IS135 will be available
delete from return_status
where issued_id = 'IS135';
-- Now Call the record of issued_id IS135 and it will show the message and the return_status will change.
-- Hence, the records are updated

select * from books
where isbn = '978-0-330-25864-8';

update books
set status = 'no'
where isbn = '978-0-330-25864-8';

select * from issued_status
where issued_book_isbn = '978-0-330-25864-8';
-- IS140

-- Calling function
call add_return_records('RS148', 'IS140', 'Good');

-- Task 15: Branch Performance Report
-- Create a query that generates a performance report for each branch,
-- showing the number of books issued, the number of books returned, 
-- and the total revenue generated from book rentals.

create table branch_performance_report as
select 
bh.branch_id,
bh.manager_id,
count(ist.issued_id) as number_books_issued,
count(rs.return_id) as number_books_returned,
sum(bk.rental_price) as total_revenue
from issued_status as ist
join employees as e
on e.emp_id = ist.issued_emp_id
join branch as bh
on e.branch_id = bh.branch_id
left join return_status as rs
on rs.issued_id = ist.issued_id
join books as bk
on bk.isbn = ist.issued_book_isbn
group by 1,2;

select * from branch_performance_report;

-- Task 16: CTAS: Create a Table of Active Members
-- Use the CREATE TABLE AS (CTAS) statement 
-- to create a new table active_members 
-- containing members who have issued at least one book in the last 1 year.
create table active_members as
select * from members
where member_id 
in(
select distinct issued_member_id
from issued_status
where issued_date >= curdate() - interval 1 year);

select * from active_members;

-- Task 17: Find Employees with the Most Book Issues Processed
-- Write a query to find the top 3 employees who have processed the most book issues. 
-- Display the employee name, number of books processed, and their branch.
select 
e.emp_name,
e.emp_id,
b.branch_id,
count(ist.issued_id) as no_books_issued
from employees as e
join branch as b
on e.branch_id = b.branch_id
join issued_status as ist
on ist.issued_emp_id = e.emp_id 
group by 1, 2
order by no_books_issued desc
limit 3;

-- Task 18: Identify Members Issuing High-Risk Books
-- Write a query to identify members who have issued books more than once 
-- with the status "damaged" in the books table. Display the member name, book title, 
-- and the number of times they've issued damaged books.
create table members_issuing_high_risk_books as
select 
m.member_name,
ist.issued_id,
r.return_id,
bc.book_title,
bc.no_issued,
bk.status,
r.book_quality
from issued_status as ist
join books as bk
on bk.isbn = ist.issued_book_isbn
join return_status as r
on r.issued_id = ist.issued_id
left join book_count as bc
on bc.book_title = bk.book_title
join members as m
on m.member_id = ist.issued_member_id;

select * from members_issuing_high_risk_books;
update members_issuing_high_risk_books
set book_quality = 'Damaged'
where no_issued > 1;

-- Task 19: Stored Procedure Objective: Create a stored procedure to manage the status of books in a library system. 
-- Description: Write a stored procedure that updates the status of a book in the library based on its issuance. 
-- The procedure should function as follows: The stored procedure should take the book_id as an input parameter. 
-- The procedure should first check if the book is available (status = 'yes'). 
-- If the book is available, it should be issued, and the status in the books table should be updated to 'no'. 
-- If the book is not available (status = 'no'), 
-- the procedure should return an error message indicating that the book is currently not available.

-- Stored Procedure
Delimiter //

create procedure manage_status(
in p_issued_id varchar(10) ,
in p_issued_member_id varchar(10),
in p_issued_book_isbn varchar(25),
in p_issued_emp_id varchar(10)
)
begin
declare
-- all the varriable
v_status varchar(10);

-- Checking if the book is available 'yes'
select status into v_status
from books
where isbn = p_issued_book_isbn;

if v_status = 'yes' then 
insert into issued_status (issued_id, issued_member_id, issued_book_isbn, issued_emp_id)
values (p_issued_id, p_issued_member_id, current_date(), p_issued_book_isbn, p_issued_emp_id);

update books
set status = 'no'
where isbn = p_issued_book_isbn;

select concat('Books record added succesfully for book isbn ; %', p_issued_book_isbn) as message;

else 
select concat('Sorry, the book with isbn ', p_issued_book_isbn, ' is unavailable') as message;

end if;

end //
Delimiter ; 

select * from books;
-- 978-0-553-29698-2 -- 'yes'
-- 978-0-375-41398-8 -- 'no'

select * from issued_status;

-- Testing procedure 
call manage_status('IS155', 'C108', '978-0-553-29698-2', 'E104');
select * from books
where isbn = '978-0-553-29698-2';

call manage_status('IS156', 'C108', '978-0-375-41398', 'E104');

-- Task 20: Create Table As Select (CTAS) Objective: 
-- Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.
-- Description: 
-- Write a CTAS query to create a new table that lists each member and the books they have issued but not returned within 60 days. 
-- The table should include: The number of overdue books. 
-- The total fines, with each day's fine calculated at $0.50. 
-- The number of books issued by each member. 
-- The resulting table should show: Member ID Number of overdue books Total fines

create table identify_day_difference as
select
ist.issued_member_id,
ist.issued_id,
ist.issued_date,
r.return_id,
r.return_date,
datediff(r.return_date, ist.issued_date) as days_difference
from issued_status as ist
join return_status as r
on r.issued_id = ist.issued_id
;

select 
issued_member_id,
(60 - days_difference) as overdue_days,
concat('$', round((60 - days_difference) * 0.50,2)) as overdue_fine
from identify_day_difference
where days_difference < 60
order by 1
;

-- Reports
-- Database Schema: Detailed table structures and relationships.
-- Data Analysis: Insights into book categories, employee salaries, member registration trends, and issued books.
-- Summary Reports: Aggregated data on high-demand books and employee performance.

-- Conclusion
-- This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.










