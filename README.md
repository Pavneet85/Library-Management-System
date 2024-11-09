# Library Management System using SQL Project

## Project Overview: 

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

## Objectives: 

    a) Set up the Library Management System Database: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
    b) CRUD Operations: Perform Create, Read, Update, and Delete operations on the data.
    c) CTAS (Create Table As Select): Utilize CTAS to create new tables based on query results.
    d) Advanced SQL Queries: Develop complex queries to analyze and retrieve specific data.

### Tools Used: SQL to obtain the results by retrieving relevant results from the respective tables with the help of SQL queries and visualizing the relevant KPIs in the form of interactive elements for better understanding. 

### Part 1: Set up the Library Management System Database - Create and populate the database with tables for branches, employees, members, books, issued status, and return status.

- Step 1 : Review the dataset which is available as csv files in excel and analyse the number of column heads and the number of rows for the data. Get familiar with the data and spend some time analysing the column heads along with data types.  

- Step 2: Load data into PostgreSQL server by creating a new database "library_db" and create tables with relevant columns along with their data types.
Do add Primary keys to each table and the resultant queries to create the 6 tables will be as follows - 

    CREATE TABLE books
    (
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price FLOAT,
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
    );

    CREATE TABLE members
    (
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(25),
            member_address VARCHAR(75),
            reg_date DATE
    );

    CREATE TABLE issued_status
    (
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10)
           
    );

    CREATE TABLE return_status
    (
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(10),
            return_book_name VARCHAR(75),
            return_date DATE,
            return_book_isbn VARCHAR(50)
         
    );

    CREATE TABLE branch
    (
            branch_id VARCHAR(10),
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
    );


    CREATE TABLE IF NOT EXISTS public.employees
    (
            emp_id VARCHAR(10),
            emp_name VARCHAR(25),
            position VARCHAR(15),
            branch_id VARCHAR(25)
    );

- Step 3: Now, in order to establish relationship between these above 6 tables, we need to develop an Entity Relationship Diagram(ERD). Right click on the library_db, and select ERD for database option. For building the relatioships, we need to alter certain tables to add Foreign Keys to develop the relationship with Primary Key of the other table. Following queries are used - 

        -- FOREIGN KEY   
        ALTER TABLE issued_status
        ADD CONSTRAINT fk_members
        FOREIGN KEY(issued_member_id)
        REFERENCES members(member_id);

        ALTER TABLE issued_status
        ADD CONSTRAINT fk_books
        FOREIGN KEY(issued_book_isbn)
        REFERENCES books(isbn);

        ALTER TABLE issued_status
        ADD CONSTRAINT fk_employees
        FOREIGN KEY(issued_emp_id)
        REFERENCES employees(emp_id);

        ALTER TABLE employees
        ADD CONSTRAINT fk_branch
        FOREIGN KEY(branch_id)
        REFERENCES branch(branch_id);   

        ALTER TABLE return_status
        ADD CONSTRAINT fk_issued_status
        FOREIGN KEY(issued_id)
        REFERENCES issued_status(issued_id);

The ERD for libraray database will look like  -
![ERD](https://github.com/user-attachments/assets/c63fc730-8a87-42fd-8c0d-40c0ef2c4982)

- Step 4: In this step, we will add data into the respective tables. This can be done by either importing the data directly using Import option or we can manually add the data using INSERT Queries. 
Cross check the data entries with Excel files by using the following sample query -

      SELECT * 
      FROM books;

### Part 2: CRUD Operations: 

a) Create a New Book Record -  

    INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
    VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

![LM 1](https://github.com/user-attachments/assets/737ad6a9-81bd-4556-b676-3c0bf66bc6eb)

b) Update an Existing Member's Address: 


    UPDATE members
    SET member_address = '125 Main St'
    WHERE member_id = 'C101';


![LM 2](https://github.com/user-attachments/assets/92f68e80-9465-45e5-9e3d-2a57565da25e)

c)  Delete a Record from the Issued Status Table - 

    DELETE FROM issued_status
    WHERE   issued_id =   'IS121';

 d) Retrieve All Books Issued by a Specific Employee  - 

    SELECT * FROM issued_status
    WHERE issued_emp_id = 'E101'

![LM 3](https://github.com/user-attachments/assets/7d39d951-264c-4c29-b4df-2b4812e1ebb3)

e) List Members Who Have Issued More Than One Book -
    
    SELECT issued_emp_id,
    COUNT(issued_id) AS total_books_issued
    FROM issued_status
    GROUP BY 1
    HAVING COUNT(issued_id) > 1;

![LM 4](https://github.com/user-attachments/assets/8c06a633-c27d-46e8-bb2f-58deee8b1b46)


f) Create Summary Tables - Used CTAS to generate new tables based on query results - each book and total book issued count - 

    CREATE TABLE bookcnts
    AS
    SELECT 
    b.isbn,
    b.book_title,
    COUNT(ist.issued_id) AS no_issued
    FROM books as b
    JOIN issued_status as ist
    ON
    ist.issued_book_isbn = b.isbn
    GROUP BY 1;

    SELECT * FROM bookcnts

![LM 5](https://github.com/user-attachments/assets/f1911d3f-7417-4973-8fd1-08aa6460216d)

### Queries to address specific questions:

g) Retrieve All Books in a Specific Category -

    SELECT * FROM books
    WHERE category = 'Classic';

![LM 6](https://github.com/user-attachments/assets/93edf036-6404-420c-811b-241370984dd9)

h) Find Total Rental Income OF Books by Category - 

    SELECT
    b.category,
    SUM(b.rental_price) AS total_rental_price,
    COUNT(*) AS total_books_issued
    FROM books b
    JOIN issued_status ist
    ON b.isbn = ist.issued_book_isbn
    GROUP BY 1
    ORDER BY 2 DESC;

![LM 7](https://github.com/user-attachments/assets/8872aedf-4076-4961-a6f4-437440244191)

i) List Members Who Registered in the Last 180 Days - 

    SELECT * FROM members
    WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
    
![LM 8](https://github.com/user-attachments/assets/980ffa67-481a-461f-9ca9-af3b5dc95a6b)

j) List Employees with Their Branch Manager's Name and their branch details – 

    SELECT 
    e1.emp_id,
    e1.emp_name,
    e1.position,
    e1.salary,
    b.*,
    e2.emp_name as manager
    FROM employees as e1
    JOIN 
    branch as b
    ON e1.branch_id = b.branch_id    
    JOIN
    employees as e2
    ON e2.emp_id = b.manager_id;

![LM 9](https://github.com/user-attachments/assets/2e3e8542-ca6f-4601-9f23-db520fa530ad)

k)  Create a Table of Books with Rental Price Above a Certain Threshold –

    CREATE TABLE expensive_books AS
    SELECT * FROM books
    WHERE rental_price > 7.00;

    SELECT * FROM expensive_books; 


![LM 10](https://github.com/user-attachments/assets/5d2fc2d8-0010-4006-b2dd-522048a44243)

l) Retrieve the List of Books Not Yet Returned - 
    
    SELECT DISTINCT ist.issued_book_name
    FROM issued_status as ist
    LEFT JOIN
    return_status as rs
    ON rs.issued_id = ist.issued_id
    WHERE rs.return_id IS NULL;

![LM 11](https://github.com/user-attachments/assets/b9e88a62-6a99-4c10-b294-d77787f81c15)


m) Identify members who have overdue books (assume a 220-day return period). Display the member's_id, member's name, book title, issue date, and days overdue - 

    SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    CURRENT_DATE - ist.issued_date AS overdues_days
    FROM issued_status AS ist
    JOIN members AS m
    ON m.member_id = ist.issued_member_id 
    JOIN books AS bk
    on bk.isbn = ist.issued_book_isbn
    LEFT JOIN return_status AS rs
    ON rs.issued_id = ist.issued_id
    WHERE rs.return_date IS NULL
    AND CURRENT_DATE - ist.issued_date > 220
    ORDER BY 1;

![LM 12](https://github.com/user-attachments/assets/f94ace1b-cdc2-4410-8349-181ecaa7b041)

n) Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals. -

    CREATE TABLE branch_reports
    AS
    SELECT 
    b.branch_id,
    b.manager_id,
    COUNT(ist.issued_id) as number_book_issued,
    COUNT(rs.return_id) as number_of_book_return,
    SUM(bk.rental_price) as total_revenue
    FROM issued_status as ist
    JOIN employees AS e
    ON e.emp_id = ist.issued_emp_id
    JOIN branch as b
    ON b.branch_id = e.branch_id
    LEFT JOIN return_status AS rs
    ON rs.issued_id = ist.issued_id
    JOIN books as bk
    ON bk.isbn = ist.issued_book_isbn
    GROUP BY 1, 2;

    SELECT * FROM branch_reports;
    

![LM 13](https://github.com/user-attachments/assets/c09d71f6-5c9f-4855-9755-998aafd7a286)

o) Create a  TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 7 months - 
  

    CREATE TABLE active_members
    AS
    SELECT * FROM members
    WHERE member_id IN
		(SELECT DISTINCT issued_member_id
		FROM issued_status
		WHERE issued_date >= CURRENT_DATE - INTERVAL'7	month');

    SELECT * FROM active_members;

![LM 14](https://github.com/user-attachments/assets/b955a663-7d32-4ed0-bc32-3f9dadf4cbb3)   

p) Find Employees with the Most Book Issues Processed - 

    SELECT 
    e.emp_name,
    b.*,
    COUNT(ist.issued_id) AS books_issued
    FROM issued_status AS ist
    JOIN employees AS e
    ON e.emp_id = ist.issued_emp_id
    JOIN branch AS b
    ON b.branch_id = e.branch_id
    GROUP BY 1, 2;

![LM 15](https://github.com/user-attachments/assets/537f098c-f506-45ba-a2a6-d80103a163e2)
