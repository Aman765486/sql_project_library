# Library Management System

A comprehensive SQL-based Library Management System that handles book tracking, member management, employee operations, and transaction processing.

## 📋 Project Overview

This system manages library operations including:
- Book inventory and categorization
- Member registration and management
- Employee and branch information
- Book issuing and return tracking
- Rental revenue calculation

## 🗄️ Database Schema

### Tables Structure

1. **branch** - Library branch information
   - branch_id (PK)
   - manager_id
   - branch_address
   - contact_no

2. **employee** - Staff information
   - emp_id (PK)
   - emp_name
   - position
   - salary
   - branch_id (FK to branch)

3. **book** - Book catalog
   - isbn (PK)
   - book_title
   - category
   - rental_price
   - status (available/issued)
   - author
   - publisher

4. **members** - Library members
   - member_id (PK)
   - member_name
   - member_address
   - reg_date

5. **issued_status** - Book issuing records
   - issued_id (PK)
   - issued_member_id (FK to members)
   - issued_book_name
   - issued_date
   - issued_book_isbn (FK to book)
   - issued_emp_id (FK to employee)

6. **return_status** - Book return records
   - return_id (PK)
   - issued_id (FK to issued_status)
   - return_book_name
   - return_date
   - return_book_isbn

## 🔧 Installation & Setup

1. **Prerequisites**
   - PostgreSQL database system
   - Basic SQL knowledge

2. **Database Setup**
   ```sql
   -- Run the complete SQL script to create database structure
   -- The script includes:
   -- - Table creation with proper constraints
   -- - Foreign key relationships
   -- - Sample data insertion
📊 Key Features
Data Management
CRUD operations for books, members, and employees

Book issuing and return tracking

Rental price management

Reporting & Analytics
Branch performance reports

Rental income analysis by category

Active member tracking

Overdue book identification

Advanced Functionality
Automated status updates on book returns

Member activity monitoring

Employee performance tracking

🚀 SQL Tasks Implemented
The project includes solutions to 17 practical SQL tasks:

Record Creation - Adding new books to inventory

Data Updates - Modifying member information

Record Deletion - Removing issued status records

Specific Queries - Finding books issued by specific employees

Aggregate Functions - Members with multiple book issues

CTAS Operations - Creating summary tables

Category Filtering - Books by specific categories

Revenue Analysis - Rental income by category

Temporal Queries - Recent member registrations

Join Operations - Employee-manager relationships

Conditional Filtering - High-value books

Status Checking - Unreturned books tracking

Overdue Management - Identifying late returns

Automated Updates - Book status synchronization

Performance Reporting - Branch-level analytics

Activity Tracking - Active member identification

Performance Analysis - Top-performing employees

📝 Sample Queries
Find Overdue Books

SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    CURRENT_DATE - ist.issued_date as overdue_days
FROM issued_status as ist
JOIN members as m ON m.member_id = ist.issued_member_id
JOIN book as bk ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status as rs ON rs.issued_id = ist.issued_id
WHERE rs.return_date IS NULL
AND (CURRENT_DATE - ist.issued_date) > 30;
Branch Performance Report

SELECT 
    b.branch_id,
    COUNT(ist.issued_id) as books_issued,
    COUNT(rs.return_id) as books_returned,
    SUM(bk.rental_price) as total_revenue
FROM issued_status as ist
JOIN employee as e ON e.emp_id = ist.issued_emp_id
JOIN branch as b ON e.branch_id = b.branch_id
LEFT JOIN return_status as rs ON rs.issued_id = ist.issued_id
JOIN book as bk ON ist.issued_book_isbn = bk.isbn
GROUP BY b.branch_id;


