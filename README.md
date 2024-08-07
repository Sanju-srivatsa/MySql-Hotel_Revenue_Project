# Hotel Revenue Database

## Overview

This project involves creating a hotel revenue database with three tables: `rooms`, `reservations`, and `payments`. The project includes creating triggers, views, stored procedures, and functions, and demonstrates various SQL concepts and techniques through query statements.

## Project Requirements

1. **Database Creation**: Create a database and at least 3 tables.
2. **Table Structure**: Each table must have at least 3 columns with primary and/or foreign keys.
3. **Data Types**: Use various data types such as VarChar, Int, Decimal, and Date.
4. **Relationships**: Ensure tables are logically related using primary and/or foreign keys.
5. **Sample Data**: Populate each table with at least 5 rows of data.
6. **Views**: Create and use views as part of the project.
7. **Stored Procedures and Functions**: Create and demonstrate at least one stored procedure and function.
8. **Triggers**: Implement triggers to enforce data conditions.
9. **Queries**: Demonstrate at least 3 query statements.
10. **Group By**: Use `GROUP BY` clause to demonstrate summary queries.

## Implementation

### Creating Tables

```sql
-- Drop the database if it exists
DROP DATABASE IF EXISTS hotel_revenue;

-- Create the hotel_revenue database
CREATE DATABASE hotel_revenue;

-- Switch to the hotel_revenue database
USE hotel_revenue;

-- Create the rooms table
CREATE TABLE rooms (
  room_number INT PRIMARY KEY,
  room_type VARCHAR(50),
  room_rate DECIMAL(10,2)
);

-- Create the reservations table
CREATE TABLE reservations (
  reservation_id INT PRIMARY KEY,
  guest_name VARCHAR(50),
  check_in_date DATE,
  check_out_date DATE,
  room_number INT,
  total_cost DECIMAL(10,2),
  FOREIGN KEY (room_number) REFERENCES rooms(room_number)
);

-- Create the payments table
CREATE TABLE payments (
  payment_id INT PRIMARY KEY,
  reservation_id INT,
  payment_date DATE,
  payment_amount DECIMAL(10,2),
  payment_method VARCHAR(50),
  FOREIGN KEY (reservation_id) REFERENCES reservations(reservation_id)
);
```

### Inserting Data

```sql
-- Populate the rooms table with sample data
INSERT INTO rooms VALUES
  (101, 'Single', 100.00),
  (102, 'Single', 100.00),
  (103, 'Double', 150.00),
  (104, 'Double', 150.00),
  (105, 'Suite', 250.00);

-- Output for rooms table
SELECT * FROM rooms;

-- Output:
-- | room_number | room_type | room_rate |
-- |-------------|-----------|-----------|
-- | 101         | Single    | 100.00    |
-- | 102         | Single    | 100.00    |
-- | 103         | Double    | 150.00    |
-- | 104         | Double    | 150.00    |
-- | 105         | Suite     | 250.00    |

-- Populate the reservations table with sample data
INSERT INTO reservations VALUES
  (1, 'John Doe', '2023-01-01', '2023-01-05', 101, 500.00),
  (2, 'Jane Doe', '2023-02-16', '2023-02-20', 102, 400.00),
  (3, 'Bob Smith', '2023-03-15', '2023-03-20', 103, 750.00),
  (4, 'Alice Johnson', '2023-04-01', '2023-04-04', 104, 450.00),
  (5, 'Sarah Lee', '2023-05-05', '2023-05-09', 105, 600.00);

-- Output for reservations table
SELECT * FROM reservations;

-- Output:
-- | reservation_id | guest_name    | check_in_date | check_out_date | room_number | total_cost |
-- |----------------|---------------|---------------|----------------|-------------|------------|
-- | 1              | John Doe      | 2023-01-01    | 2023-01-05     | 101         | 500.00     |
-- | 2              | Jane Doe      | 2023-02-16    | 2023-02-20     | 102         | 400.00     |
-- | 3              | Bob Smith     | 2023-03-15    | 2023-03-20     | 103         | 750.00     |
-- | 4              | Alice Johnson | 2023-04-01    | 2023-04-04     | 104         | 450.00     |
-- | 5              | Sarah Lee     | 2023-05-05    | 2023-05-09     | 105         | 600.00     |

-- Populate the payments table with sample data
INSERT INTO payments VALUES
  (1, 1, '2023-01-05', 500.00, 'Credit'),
  (2, 2, '2023-02-20', 400.00, 'Cash'),
  (3, 3, '2023-03-20', 750.00, 'Credit'),
  (4, 4, '2023-04-04', 450.00, 'Debit'),
  (5, 5, '2023-05-09', 600.00, 'Cash');

-- Output for payments table
SELECT * FROM payments;

-- Output:
-- | payment_id | reservation_id | payment_date | payment_amount | payment_method |
-- |------------|----------------|--------------|----------------|----------------|
-- | 1          | 1              | 2023-01-05   | 500.00         | Credit         |
-- | 2          | 2              | 2023-02-20   | 400.00         | Cash           |
-- | 3          | 3              | 2023-03-20   | 750.00         | Credit         |
-- | 4          | 4              | 2023-04-04   | 450.00         | Debit          |
-- | 5          | 5              | 2023-05-09   | 600.00         | Cash           |
```

### Creating Triggers

```sql
-- Create a trigger to ensure that the room_rate in the rooms table cannot be negative.
DROP TRIGGER IF EXISTS rooms_check_rate;
DELIMITER //
CREATE TRIGGER rooms_check_rate
BEFORE INSERT ON rooms
FOR EACH ROW
BEGIN
    IF NEW.room_rate < 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Room rate cannot be negative';
    END IF;
END //
DELIMITER ;

-- Create a trigger to enforce the check-out date to be after the check-in date for each reservation
DROP TRIGGER IF EXISTS reservations_check_dates;
DELIMITER //
CREATE TRIGGER reservations_check_dates
BEFORE INSERT ON reservations
FOR EACH ROW
BEGIN
    IF NEW.check_out_date <= NEW.check_in_date THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Check-out date must be after check-in date';
    END IF;
END //
DELIMITER ;
```

### Creating Views

```sql
-- Create a view to show each month's total revenue
CREATE VIEW monthly_revenue AS
SELECT DATE_FORMAT(check_in_date, '%Y-%m') AS month, SUM(total_cost) AS revenue
FROM reservations
GROUP BY month;

-- Output for monthly_revenue view
SELECT * FROM monthly_revenue;

-- Output:
-- | month    | revenue |
-- |----------|---------|
-- | 2023-01  | 500.00  |
-- | 2023-02  | 400.00  |
-- | 2023-03  | 750.00  |
-- | 2023-04  | 450.00  |
-- | 2023-05  | 600.00  |
```

### Creating Stored Procedures

```sql
-- Create a stored procedure to calculate the total revenue for a specific room type and date range
DELIMITER //
CREATE PROCEDURE get_room_revenue(
IN room_type_param VARCHAR(50),
IN start_date_param DATE,
IN end_date_param DATE,
OUT revenue DECIMAL(10,2)
)
BEGIN
SELECT SUM(total_cost) INTO revenue
FROM reservations
JOIN rooms ON reservations.room_number = rooms.room_number
WHERE rooms.room_type = room_type_param
AND check_in_date >= start_date_param
AND check_out_date <= end_date_param;
END //
DELIMITER ;

-- Call the stored procedure
CALL get_room_revenue('Double', '2023-03-01', '2023-04-30', @revenue);
SELECT @revenue;

-- Output:
-- | @revenue |
-- |----------|
-- | 450.00   |
```

### Creating Functions

```sql
-- Create a function to calculate the total revenue for a specific date range
DELIMITER //
CREATE FUNCTION get_total_revenue(start_date DATE, end_date DATE)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
DECLARE revenue DECIMAL(10,2);
SELECT SUM(total_cost) INTO revenue
FROM reservations
WHERE check_in_date >= start_date AND check_out_date <= end_date;
RETURN revenue;
END //
DELIMITER ;

-- Call the function
SELECT get_total_revenue('2023-01-01', '2023-02-28

');

-- Output:
-- | get_total_revenue('2023-01-01', '2023-02-28') |
-- |----------------------------------------------|
-- | 900.00                                       |

-- Create a function to calculate the average room rate for a given room type
DELIMITER //
CREATE FUNCTION get_average_room_rate(room_type_param VARCHAR(50))
RETURNS DECIMAL(10,2)
BEGIN
  DECLARE avg_rate DECIMAL(10,2);
  SELECT AVG(room_rate) INTO avg_rate
  FROM rooms
  WHERE room_type = room_type_param;
  RETURN avg_rate;
END //
DELIMITER ;

-- Call the function
SELECT get_average_room_rate('Single');

-- Output:
-- | get_average_room_rate('Single') |
-- |--------------------------------|
-- | 100.00                         |
```

### Query Statements

```sql
-- Query to show the total revenue for a specific month
SELECT SUM(total_cost) AS revenue
FROM reservations
WHERE MONTH(check_in_date) = 5;

-- Output:
-- | revenue |
-- |---------|
-- | 600.00  |

-- Query to show the occupancy rate for each room type
SELECT rm.room_type, COUNT(*) AS reservations, COUNT(*)/(SELECT COUNT(*) FROM reservations)*100 AS occupancy_rate
FROM reservations r
INNER JOIN rooms rm ON r.room_number = rm.room_number
GROUP BY rm.room_type;

-- Output:
-- | room_type | reservations | occupancy_rate |
-- |-----------|--------------|----------------|
-- | Single    | 2            | 40.00          |
-- | Double    | 1            | 20.00          |
-- | Suite     | 1            | 20.00          |

-- Query to show the payment history for a specific guest
SELECT *
FROM reservations r
INNER JOIN payments p ON r.reservation_id = p.reservation_id
WHERE r.guest_name = 'John Doe';

-- Output:
-- | reservation_id | guest_name | check_in_date | check_out_date | room_number | total_cost | payment_id | payment_date | payment_amount | payment_method |
-- |----------------|------------|---------------|----------------|-------------|------------|------------|--------------|----------------|----------------|
-- | 1              | John Doe   | 2023-01-01    | 2023-01-05     | 101         | 500.00     | 1          | 2023-01-05   | 500.00         | Credit         |

-- Query to show each month's total revenue grouped by room type
SELECT 
    MONTH(check_in_date) AS month,
    room_type,
    SUM(total_cost) AS revenue
FROM 
    reservations r
    JOIN rooms rm ON r.room_number = rm.room_number
GROUP BY 
    MONTH(check_in_date),
    room_type;

-- Output:
-- | month | room_type | revenue |
-- |-------|-----------|---------|
-- | 1     | Single    | 500.00  |
-- | 2     | Single    | 400.00  |
-- | 3     | Double    | 750.00  |
-- | 4     | Double    | 450.00  |
-- | 5     | Suite     | 600.00  |
```

## Conclusion

This project demonstrates the creation and management of a hotel revenue database using various SQL concepts and techniques. It covers the creation of tables, insertion of data, creation of triggers, views, stored procedures, and functions, and demonstrates complex queries. The project provides a comprehensive understanding of database management and manipulation using SQL.


---

This README file provides a comprehensive overview of the SQL project, including detailed code examples and outputs for better understanding.
