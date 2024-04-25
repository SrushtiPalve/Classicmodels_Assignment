USE classicmodels;

-- DAY 3 
                                            
/* 1) Show customer number, customer name, state and credit limit from customers table for below conditions. 
		Sort the results by highest to lowest values of creditLimit.
		● State should not contain null values
		● credit limit should be between 50000 and 100000      
*/

SELECT  customerNumber, customerName, state, creditLimit
FROM customers
WHERE state IS NOT NULL
AND creditLimit BETWEEN 50000 AND 100000
ORDER BY creditLimit DESC;

/* 2) Show the unique productline values containing the word cars at the end from products table.
*/

SELECT DISTINCT productLine 
FROM products
WHERE productLine LIKE "%cars";

-- DAY 4                                                
/* 1) Show the orderNumber, status and comments from orders table for shipped status only. 
	If some comments are having null values then show them as “-“.
*/

SELECT orderNumber, status, IFNULL(comments, "-") AS Comments
FROM orders
WHERE status = "Shipped";
             #OR
SELECT orderNumber, status, COALESCE(comments, "-") AS Comments
FROM orders
WHERE status = "Shipped";

/*2) Select employee number, first name, job title and job title abbreviation from employees table based on following conditions. 
		If job title is one among the below conditions, then job title abbreviation column should show below forms. 
		● President then “P” 
		● Sales Manager / Sale Manager then “SM” 
		● Sales Rep then “SR” 
		● Containing VP word then “VP”
*/

SELECT employeeNumber, firstName, jobTitle,
CASE
	WHEN jobTitle LIKE "President" THEN "P"
    WHEN jobTitle LIKE "%Manager%" THEN "SM"
    WHEN jobTitle LIKE "Sales Rep" THEN "SR"
    WHEN jobTitle LIKE "%VP%" THEN "VP"
    ELSE "-"
END    AS JobTitleAbbreviation
FROM employees;

-- DAY 5                                       
/* 1) For every year, find the minimum amount value from payments table
*/

SELECT 
	DISTINCT(YEAR(paymentDate)) AS Year, 
	MIN(amount) Min_Amount
FROM payments
GROUP BY YEAR(paymentDate)
ORDER BY YEAR(paymentDate);   

/*2) For every year and every quarstudentster, find the unique customers and total orders from orders table. 
	Make sure to show the quarter as Q1,Q2 etc.
*/

SELECT YEAR(orderDate) AS Year, 
	CONCAT("Q", QUARTER(orderDate)) Quarter, 
    COUNT(DISTINCT customerNumber) AS Unique_Customers,
    COUNT(orderNumber) AS Total_Orders
FROM orders
GROUP BY Year,Quarter; 

/*3) Show the formatted amount in thousands unit (e.g. 500K, 465K etc.) for every month (e.g. Jan, Feb etc.) 
with filter on total amount as 500000 to 1000000. 
Sort the output by total amount in descending mode. [ Refer. Payments Table] 
*/

SELECT 
	DATE_FORMAT(paymentDate, '%b') AS Month,
    CONCAT(FORMAT(SUM(amount)/1000,0),'K') as Formatted_Amount
FROM payments
GROUP BY Month
HAVING SUM(amount) BETWEEN 500000 AND 1000000
ORDER BY SUM(amount) DESC;


-- DAY 6
/*1) Create a journey table with following fields and constraints.
	● Bus_ID (No null values)
	● Bus_Name (No null values)
	● Source_Station (No null values)
	● Destination (No null values)
	● Email (must not contain any duplicates)
*/

CREATE TABLE journey(
	Bus_ID INT NOT NULL,
    Bus_Name VARCHAR(100) NOT NULL,
    Source_Station VARCHAR(100) NOT NULL,
    Destination VARCHAR(100) NOT NULL,
    Email VARCHAR (100) UNIQUE
);
SELECT * FROM journey;
DESCRIBE journey;

/*2) Create vendor table with following fields and constraints.
	● Vendor_ID (Should not contain any duplicates and should not be null)
	● Name (No null values)
	● Email (must not contain any duplicates)
	● Country (If no data is available then it should be shown as “N/A”)
*/

CREATE TABLE vendor(
Vendor_ID INT UNIQUE NOT NULL,
Name VARCHAR(100) NOT NULL,
Email VARCHAR(100) UNIQUE,
Country VARCHAR(100) DEFAULT "N/A"
);
SELECT * FROM vendor;
DESCRIBE vendor;

/*3) Create movies table with following fields and constraints.
	● Movie_ID (Should not contain any duplicates and should not be null)
	● Name (No null values)
	● Release_Year (If no data is available then it should be shown as “-”)
	● Cast (No null values)
	● Gender (Either Male/Female)
	● No_of_shows (Must be a positive number)
*/

CREATE TABLE movies(
Movie_ID INT PRIMARY KEY,
Name VARCHAR(100) NOT NULL,
Release_Year VARCHAR(10) DEFAULT "-",
Cast VARCHAR (100) NOT NULL,
Gender ENUM("Male","Female"),
No_of_shows INT CHECK(No_of_shows>=0)
);
SELECT * FROM movies;
DESCRIBE movies;

/*4)Create the following tables. Use auto increment wherever applicable
a. Product
✔	product_id - primary key
✔	product_name - cannot be null and only unique values are allowed
✔	description
✔	supplier_id - foreign key of supplier table

b. Suppliers
✔	supplier_id - primary key
✔	supplier_name
✔	location

c. Stock
✔	id - primary key
✔	product_id - foreign key of product table
✔	balance_stock
*/

CREATE TABLE Suppliers(
Supplier_ID INT AUTO_INCREMENT PRIMARY KEY,
Supplier_Name VARCHAR(100) NOT NULL,
Location VARCHAR(100)
);
SELECT * FROM suppliers;
DESCRIBE suppliers;

CREATE TABLE Product(
Product_ID INT AUTO_INCREMENT PRIMARY KEY,
Product_Name VARCHAR (100) NOT NULL UNIQUE,
Description TEXT,
Supplier_ID INT,
FOREIGN KEY (Supplier_ID) REFERENCES Suppliers(Supplier_ID)
);
SELECT * FROM product;
DESCRIBE product;

CREATE TABLE Stock(
Stock_ID INT PRIMARY KEY,
Product_ID INT,
FOREIGN KEY (Product_ID) REFERENCES Product(Product_ID),
Balance_Stock INT
);
SELECT * FROM stock;
DESCRIBE stock;

-- DAY 7                                               
/* 1) Show employee number, Sales Person (combination of first and last names of employees), 
	unique customers for each employee number and sort the data by highest to lowest unique customers.
Tables: Employees, Customers
*/

SELECT * FROM employees;
SELECT * FROM customers;
SELECT e.employeeNumber,
	CONCAT(e.firstName," ",e.lastName) AS "Sales Person", 
    COUNT(c.customerName) AS "Unique Customer"
FROM employees e
JOIN customers c ON e.employeeNumber = c.salesRepEmployeeNumber
GROUP BY e.employeeNumber, CONCAT(e.firstName," ",e.lastName)
ORDER BY COUNT(c.customerName) DESC;


/*2) Show total quantities, total quantities in stock (produts), left over quantities for each product and each customer. 
		Sort the data by customer number.
Tables: Customers, Orders, Orderdetails, Products
*/

SELECT * FROM customers;
SELECT * FROM orders;
SELECT * FROM orderDetails;
SELECT * FROM products;

SELECT 
	c.customerNumber AS Customer_Number, 
    c.customerName AS Customer_Name, 
    p.productCode As Product_Code, 
    p.productName AS Product_Name,
    SUM(od.quantityOrdered) AS Ordered_Quantity,
    p.quantityInStock AS Total_Investory,
    IFNULL(p.quantityInStock - SUM(od.quantityOrdered),0) AS Left_Quantity
FROM customers c 
JOIN orders o 
	ON c.customerNumber = o.customerNumber
JOIN orderDetails od 
	ON o.orderNumber = od.orderNumber
LEFT JOIN products p 
	ON  od.productCode = p.productCode
GROUP BY Customer_Number, Customer_Name, Product_Code,  Product_Name, Total_Investory
ORDER BY  Customer_Number;

/*3) Create below tables and fields. (You can add the data as per your wish)
●	Laptop: (Laptop_Name)
●	Colours: (Colour_Name)
Perform cross join between the two tables and find number of rows
*/

CREATE TABLE Laptop(
Laptop_Name VARCHAR(15)
);

CREATE TABLE Colour(
Colour_Name VARCHAR(15)
);

INSERT INTO Laptop
VALUES ("Dell"), ("HP");
SELECT * FROM Laptop;

INSERT INTO Colour
VALUES ("White"), ("Silver"), ("Black");
SELECT * FROM Colour;

SELECT l.Laptop_Name, c.Colour_Name 
FROM Laptop l
CROSS JOIN Colour c
ORDER BY l.Laptop_Name;
-- 6 rows (2*3)

/*4) Create table project with below fields.
●	EmployeeID
●	FullName
●	Gender
●	ManagerID
Add given data into it.
Find out the names of employees and their related managers
*/

CREATE TABLE Project(
Emp_ID INT NOT NULL UNIQUE AUTO_INCREMENT,
Full_Name VARCHAR(20),
Gender ENUM("Male", "Female"),
Manager_ID INT
);

INSERT INTO Project (Full_Name, Gender, Manager_ID)
VALUES
('Pranaya', 'Male', 3),
('Priyanka', 'Female', 1),
('Preety', 'Female', NULL),
('Anurag', 'Male', 1),
('Sambit', 'Male', 1),
('Rajesh', 'Male', 3),
('Hina', 'Female', 3);

SELECT * FROM Project;
SELECT manager.Full_Name AS Manager_Name,
	employee.Full_Name AS Emp_Name
FROM  Project as manager
JOIN  Project as employee on manager.Emp_ID = employee.Manager_ID
ORDER BY Manager_Name;

-- DAY 8                                                
/*Create table facility. Add the below fields into it.
●	Facility_ID
●	Name
●	State
●	Country
i) Alter the table by adding the primary key and auto increment to Facility_ID column.
ii) Add a new column city after name with data type as varchar which should not accept any null values.
*/

CREATE TABLE Facility(
Facility_ID INT,
Name VARCHAR(100),
State VARCHAR(100),
Country VARCHAR(100)
);

ALTER TABLE Facility
MODIFY COLUMN Facility_ID INT PRIMARY KEY AUTO_INCREMENT;

ALTER TABLE Facility
ADD COLUMN City VARCHAR(100) NOT NULL AFTER Name;

DESCRIBE Facility;

-- DAY 9
/*Create table university with below fields.
●	ID
●	Name
Add the data into it as it is.
Remove the spaces from everywhere and update the column like Pune University etc
*/

CREATE TABLE university (
ID INT PRIMARY KEY AUTO_INCREMENT,
Name VARCHAR(100)
);


INSERT INTO university (Name)
VALUES 
("       Pune          University     "), 
("  Mumbai          University     "),
("     Delhi   University     "),
("Madras University"),
("Nagpur University");

SELECT * FROM university;

UPDATE university
SET Name = REPLACE(TRIM(Name), 'University', ' University')
WHERE Name LIKE '%University';

-- DAY 10                                                
/*Create the view products status. Show year wise total products sold. 
Also find the percentage of total value for each year. 
The output should look as shown in below figure.
*/

CREATE VIEW products_status AS
SELECT
	YEAR(o.orderDate) AS Year,
    CONCAT(COUNT(od.quantityOrdered),
        " (",
		ROUND((SUM(od.quantityOrdered*od.priceEach)/ SUM(SUM(od.quantityOrdered*od.priceEach)) OVER()) * 100),
        "%)"
	) AS "Value"
FROM orders o
JOIN orderdetails od 
	ON o.orderNumber = od.orderNumber
GROUP BY Year
ORDER BY COUNT(o.orderDate) DESC;

SELECT * FROM products_status ;

-- DAY 11
/*1) Create a stored procedure GetCustomerLevel which takes input as customer number 
and gives the output as either Platinum, Gold or Silver as per below criteria.
Table: Customers
●	Platinum: creditLimit > 100000
●	Gold: creditLimit is between 25000 to 100000
●	Silver: creditLimit < 25000
*/

select * from customers;
DELIMITER //

CREATE PROCEDURE GetCustomerLevel (IN customer_number INT)

BEGIN
	DECLARE Credit INT;
    SELECT creditLimit INTO Credit
    FROM customers
    WHERE customerNumber = customer_number ;

	CASE
		WHEN Credit > 100000 THEN SELECT "Platinum";
        WHEN Credit > 25000 THEN SELECT "Gold";
        ELSE SELECT "Silver";
    END CASE;
END//

DELIMITER ;

CALL GetCustomerLevel(141);
CALL GetCustomerLevel(471);
CALL GetCustomerLevel(103);


/*2) Create a stored procedure Get_country_payments which takes in year and country as inputs and gives year wise, 
country wise total amount as an output. Format the total amount to nearest thousand unit (K) 
Tables: Customers, Payments
*/

select * from customers;
select * from payments;

DELIMITER //

CREATE PROCEDURE Get_country_payments (
				IN Input_Year INT, 
                IN Input_Country VARCHAR(20))
BEGIN
	SELECT 
		YEAR(p.paymentDate) AS Year,
		c.country AS Country,
		CONCAT(FORMAT(SUM(p.amount)/1000,0), "K") AS "Total Amount"
    FROM payments p
    JOIN customers c
		ON p.customerNumber = c.customerNumber
	
    WHERE YEAR(p.paymentDate) = Input_Year
		AND c.country = Input_Country 
	
    GROUP BY Year, Country;
END //

DELIMITER ;

-- DAY 12                                                
/*1) Calculate year wise, month name wise count of orders and year over year (YoY) percentage change. 
Format the YoY values in no decimals and show in % sign.
Table: Orders
*/

WITH CTE AS (
SELECT 
	YEAR(orderDate) AS Year,
    MONTHNAME(orderDate) AS Month,
    COUNT(orderDate) AS Order_Count
FROM orders
GROUP BY Year, Month
)
SELECT 
	Year,
    Month,
    Order_Count as "Total Orders",
    CONCAT(ROUND((100 * ((Order_Count)- LAG(Order_Count) OVER (ORDER BY Year))/ (LAG(Order_Count) OVER (ORDER BY Year))),0), " %")
    AS "% YoY Change"
FROM CTE;
    

/*2)	Create the table emp_udf with below fields.
●	Emp_ID
●	Name
●	DOB
Add the data as shown in below query.
INSERT INTO Emp_UDF (Name, DOB)
VALUES ("Piyush", "1990-03-30"), ("Aman", "1992-08-15"), ("Meena", "1998-07-28"), ("Ketan", "2000-11-21"), ("Sanjay", "1995-05-21");
Create a user defined function calculate_age which returns the age in years and months (e.g. 30 years 5 months) by accepting DOB column as a parameter.
*/

 CREATE TABLE Emp_UDF (
		Emp_ID INT PRIMARY KEY AUTO_INCREMENT,
        Name VARCHAR(100),
        DOB DATE);

INSERT INTO Emp_UDF (Name, DOB)
VALUES 
("Piyush", "1990-03-30"), 
("Aman", "1992-08-15"), 
("Meena", "1998-07-28"), 
("Ketan", "2000-11-21"), 
("Sanjay", "1995-05-21");

SELECT * FROM Emp_UDF;

DELIMITER //

CREATE FUNCTION calculate_age(date_of_birth DATE)
RETURNS VARCHAR(50)
DETERMINISTIC
BEGIN
	DECLARE years INT;
	DECLARE months INT;
 	DECLARE age VARCHAR(50);

SET years = TIMESTAMPDIFF(YEAR, date_of_birth, CURDATE());
SET months = TIMESTAMPDIFF(MONTH, date_of_birth, CURDATE()) % 12;

SET age = CONCAT( years, " years ", months, " months");

RETURN age;
END //

DELIMITER ;

SELECT Emp_ID, Name, DOB, calculate_age(DOB) AS Age 
FROM Emp_UDF;

-- DAY 13                                                
/*1) Display the customer numbers and customer names from customers table who have not placed any orders using subquery
Table: Customers, Orders
*/

SELECT * FROM customers;
SELECT * FROM orders;

SELECT c.customerNumber, c.customerName
FROM customers c
WHERE c.customerNumber NOT IN (SELECT o.customerNumber FROM orders o);


/*2) Write a full outer join between customers and orders using union 
and get the customer number, customer name, count of orders for every customer. (15-02-2024 – 12:45:00)
Table: Customers, Orders
*/

SELECT * FROM customers;
SELECT * FROM orders;

SELECT c.customerNumber, c.customerName, COUNT(o.orderNumber) AS Total_Orders 
FROM customers c
LEFT JOIN orders o
	ON (c.customerNumber = o.customerNumber)
GROUP BY c.customerNumber, c.customerName

UNION

SELECT c.customerNumber, c.customerName, COUNT(o.orderNumber) AS Total_Orders  FROM customers c
RIGHT JOIN orders o
ON (c.customerNumber = o.customerNumber)
GROUP BY c.customerNumber, c.customerName;

/*3) Show the second highest quantity ordered value for each order number.
Table: Orderdetails
*/

SELECT ordernumber, MAX(quantityOrdered) as quantityOrdered
FROM orderdetails o
WHERE quantityOrdered < 
	(SELECT MAX(quantityOrdered) 
		FROM orderdetails as od
		WHERE od.ordernumber = o.ordernumber)
GROUP BY o.ordernumber;


/*4) For each ordernumber count the number of products and then find the min and max of the values among count of orders.
Table: Orderdetails
*/

SELECT * FROM orderdetails;
SELECT 
	MAX(product_count) AS "Max(Total)",
    MIN(product_count) AS "Min(Total)"
FROM (SELECT orderNumber,
		count(*) AS product_count
	  FROM orderdetails
      GROUP BY orderNumber)
      AS Counts;

/*5) Find out how many product lines are there for which the buy price value is greater than the average of buy price value. 
Show the output as product line and its count.
*/

SELECT * FROM products;

SELECT 
	productLine,
	COUNT(*) AS Total
FROM products
WHERE buyPrice >
	(SELECT AVG(buyPrice)
		FROM products) 
GROUP BY productLine
ORDER BY Total DESC;


-- DAY 14
/*Create the table Emp_EH. Below are its fields.
●	EmpID (Primary Key)
●	EmpName
●	EmailAddress
Create a procedure to accept the values for the columns in Emp_EH. 
Handle the error using exception handling concept. 
Show the message as “Error occurred” in case of anything wrong.
*/

 DELIMITER //
 
 CREATE PROCEDURE Insert_Emp_EH (
				IN p_EmpID INT,
                IN p_EmpName VARCHAR (100),
                IN p_EmailAddress VARCHAR (100)
                )
BEGIN
	DECLARE EXIT HANDLER FOR SQLEXCEPTION
	BEGIN 
		ROLLBACK;
        SELECT "Error Occurred" AS Message;
	END;
    
    START TRANSACTION;
    INSERT INTO Emp_EH (EmpID, EmpName, EmailAddress)
    VALUES (p_EmpID, p_EmpName, p_EmailAddress);
    
	COMMIT;
    
    SELECT "Values Inserted succesfully" AS Message;
END //

DELIMITER ;

-- DAY 15                                               
/*Create the table Emp_BIT. Add below fields in it.
●	Name
●	Occupation
●	Working_date
●	Working_hours
Insert the data as shown in below query.
Create before insert trigger to make sure any new value of Working_hours, if it is negative, then it should be inserted as positive.
*/

CREATE TABLE Emp_BIT (
	Name VARCHAR(100),
    Occupation VARCHAR(100),
    Working_Date DATE,
    Working_Hours INT);

INSERT INTO Emp_BIT VALUES
('Robin', 'Scientist', '2020-10-04', 12),  
('Warner', 'Engineer', '2020-10-04', 10),  
('Peter', 'Actor', '2020-10-04', 13),  
('Marco', 'Doctor', '2020-10-04', 14),  
('Brayden', 'Teacher', '2020-10-04', 12),  
('Antonio', 'Business', '2020-10-04', 11);  
SELECT * FROM Emp_BIT ;

DELIMITER //

CREATE TRIGGER positive_working_hours
BEFORE INSERT 
ON Emp_BIT
FOR EACH ROW
BEGIN 
	IF New.Working_Hours < 0 THEN 
    SET New.Working_Hours  = - New.Working_Hours;
	END IF;
END //

DELIMITER ;
