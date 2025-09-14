# week8-database-assigment-studentrecord
Week 8_studentrecord_database assignment
-- student_records_db.sql
-- Database: Student Records Management System
-- Author: (your name)
-- Date: (today)

-- 1. Create Database
DROP DATABASE IF EXISTS studentrecordsdb;
CREATE DATABASE studentrecordsdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE studentrecordsdb;

-- 2. Create Tables

-- Schools (exam centres / schools)
CREATE TABLE schools (
  school_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  address VARCHAR(500),
  city VARCHAR(100),
  state VARCHAR(100),
  phone VARCHAR(50),
  email VARCHAR(150),
  UNIQUE KEY uq_school_name_city (name, city)
) ENGINE=InnoDB;

-- Subjects (GCE Subjects / Course codes)
CREATE TABLE subjects (
  subject_id INT AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(10) NOT NULL,
  name VARCHAR(255) NOT NULL,
  level ENUM('OL','AL') NOT NULL,
  UNIQUE KEY uq_subject_code_level (code, level)
) ENGINE=InnoDB;

-- Students
CREATE TABLE students (
  student_id INT AUTO_INCREMENT PRIMARY KEY,
  registration_no VARCHAR(30) NOT NULL UNIQUE, -- e.g., 2025OL12345 or custom id
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  gender ENUM('F','M','Other') DEFAULT 'F',
  dob DATE,
  school_id INT,
  centre_number VARCHAR(20),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (school_id) REFERENCES schools(school_id) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Many-to-many: enrollments / registrations (which student registered for which subject)
CREATE TABLE enrollments (
  enrollment_id INT AUTO_INCREMENT PRIMARY KEY,
  student_id INT NOT NULL,
  subject_id INT NOT NULL,
  exam_year YEAR NOT NULL,
  exam_level ENUM('OL','AL') NOT NULL,
  registration_date DATE DEFAULT (CURRENT_DATE),
  UNIQUE KEY uq_student_subject_year (student_id, subject_id, exam_year),
  FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE ON UPDATE CASCADE,
  FOREIGN KEY (subject_id) REFERENCES subjects(subject_id) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Results (store grades per enrollment)
CREATE TABLE results (
  result_id INT AUTO_INCREMENT PRIMARY KEY,
  enrollment_id INT NOT NULL,
  grade VARCHAR(5) NOT NULL, -- e.g., A, B, C, D, E, F, U
  remarks VARCHAR(255),
  recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (enrollment_id) REFERENCES enrollments(enrollment_id) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Payments (payment for registration)
CREATE TABLE payments (
  payment_id INT AUTO_INCREMENT PRIMARY KEY,
  student_id INT NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  currency CHAR(3) DEFAULT 'NGN',
  method ENUM('Bank','MTN','Orange','Camtel','Other') DEFAULT 'Bank',
  transaction_ref VARCHAR(100) UNIQUE,
  status ENUM('Pending','Completed','Failed') DEFAULT 'Pending',
  paid_at DATETIME,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Admin users (for portal)
CREATE TABLE admins (
  admin_id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(100) NOT NULL UNIQUE,
  full_name VARCHAR(200),
  email VARCHAR(150) UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  role ENUM('admin','registrar','viewer') DEFAULT 'viewer',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

-- Audit log (optional)
CREATE TABLE audit_logs (
  audit_id BIGINT AUTO_INCREMENT PRIMARY KEY,
  admin_id INT,
  action VARCHAR(255),
  details TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (admin_id) REFERENCES admins(admin_id) ON DELETE SET NULL
) ENGINE=InnoDB;

-- 3. Insert sample data (test records)
-- Schools
INSERT INTO schools (name, address, city, state, phone, email) VALUES
('Government Girls Secondary School Yaounde', 'Mvog-Betsi', 'Yaounde', 'Capital city', '+237680629093', 'lilexbih@yahoo.com'),
('Hope Secondary School', 'Plot 10, Outskirts Rd', 'Yaounde', 'Capital city', '+237680538979', 'numforl22@gmail.com');

-- Subjects (a subset)
INSERT INTO subjects (code, name, level) VALUES
('0530','English Language','OL'),
('0545','French','OL'),
('0570','Mathematics','OL'),
('0510','Biology','OL'),
('0515','Chemistry','OL'),
('0580','Physics','OL'),
('0705','Accounting','AL'),
('0710','Biology','AL'),
('0780','Physics','AL'),
('0795','Computer Science','AL');

-- Students
INSERT INTO students (registration_no, first_name, last_name, gender, dob, school_id, centre_number) VALUES
('2025OL10001','Aisha','Sule','F','2009-05-14',1,'11234'),
('2025OL10002','Grace','Okoro','F','2009-11-02',1,'11234'),
('2025AL20001','Rita','Ibrahim','F','2006-03-22',2,'21012');

-- Enrollments (students choose subjects)
-- Aisha selects English, French, Mathematics, Biology (4 subjects)
INSERT INTO enrollments (student_id, subject_id, exam_year, exam_level) VALUES
(1, (SELECT subject_id FROM subjects WHERE code='0530' AND level='OL'), 2025, 'OL'),
(1, (SELECT subject_id FROM subjects WHERE code='0545' AND level='OL'), 2025, 'OL'),
(1, (SELECT subject_id FROM subjects WHERE code='0570' AND level='OL'), 2025, 'OL'),
(1, (SELECT subject_id FROM subjects WHERE code='0510' AND level='OL'), 2025, 'OL');

-- Grace selects English, Mathematics, Chemistry, Physics
INSERT INTO enrollments (student_id, subject_id, exam_year, exam_level) VALUES
(2, (SELECT subject_id FROM subjects WHERE code='0530' AND level='OL'), 2025, 'OL'),
(2, (SELECT subject_id FROM subjects WHERE code='0570' AND level='OL'), 2025, 'OL'),
(2, (SELECT subject_id FROM subjects WHERE code='0515' AND level='OL'), 2025, 'OL'),
(2, (SELECT subject_id FROM subjects WHERE code='0580' AND level='OL'), 2025, 'OL');

-- Rita (A/L) selects Biology, Physics, Computer Science
INSERT INTO enrollments (student_id, subject_id, exam_year, exam_level) VALUES
(3, (SELECT subject_id FROM subjects WHERE code='0710' AND level='AL'), 2025, 'AL'),
(3, (SELECT subject_id FROM subjects WHERE code='0780' AND level='AL'), 2025, 'AL'),
(3, (SELECT subject_id FROM subjects WHERE code='0795' AND level='AL'), 2025, 'AL');

-- Results
INSERT INTO results (enrollment_id, grade, remarks) VALUES
((SELECT enrollment_id FROM enrollments WHERE student_id=1 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0530')),'B',''),
((SELECT enrollment_id FROM enrollments WHERE student_id=1 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0545')),'C',''),
((SELECT enrollment_id FROM enrollments WHERE student_id=1 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0570')),'E',''),
((SELECT enrollment_id FROM enrollments WHERE student_id=1 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0510')),'D',''),

((SELECT enrollment_id FROM enrollments WHERE student_id=2 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0530')),'A',''),
((SELECT enrollment_id FROM enrollments WHERE student_id=2 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0570')),'C',''),
((SELECT enrollment_id FROM enrollments WHERE student_id=2 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0515')),'U',''),
((SELECT enrollment_id FROM enrollments WHERE student_id=2 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0580')),'B',''),

((SELECT enrollment_id FROM enrollments WHERE student_id=3 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0710')),'C',''),
((SELECT enrollment_id FROM enrollments WHERE student_id=3 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0780')),'D',''),
((SELECT enrollment_id FROM enrollments WHERE student_id=3 AND subject_id=(SELECT subject_id FROM subjects WHERE code='0795')),'A','');

-- Payments
INSERT INTO payments (student_id, amount, currency, method, transaction_ref, status, paid_at) VALUES
(1, 5000.00, 'CFA', 'MTN', 'MTN-2025-0001', 'Completed', '2025-07-01 10:00:00'),
(2, 5000.00, 'CFA', 'Bank', 'BANK-2025-0002', 'Completed', '2025-07-02 11:30:00'),
(3, 7000.00, 'CFA', 'Orange', 'ORNG-2025-0003', 'Pending', NULL);

-- Admin user (password hash placeholder)
INSERT INTO admins (username, full_name, email, password_hash, role) VALUES
('superadmin','Dr Linda Numfor','numforl22@gmail.com','$2b$12$examplehash', 'admin');

-- 4. Useful views and queries

-- View: student result summary (calc pass/fail per student per year)
DROP VIEW IF EXISTS vw_student_summary;
CREATE VIEW vw_student_summary AS
SELECT s.student_id, s.registration_no, CONCAT(s.first_name,' ',s.last_name) AS student_name,
       s.school_id,
       e.exam_year,
       COUNT(r.result_id) AS total_subjects,
       SUM(CASE WHEN r.grade IN ('A','B','C','D','E') THEN 1 ELSE 0 END) AS passed_count,
       SUM(CASE WHEN r.grade NOT IN ('A','B','C','D','E') THEN 1 ELSE 0 END) AS failed_count
FROM students s
JOIN enrollments e ON e.student_id = s.student_id
LEFT JOIN results r ON r.enrollment_id = e.enrollment_id
GROUP BY s.student_id, e.exam_year;

-- View: students with pass/fail status for OL (rules: >=4 passes and must include English(0530), French(0545), Mathematics(0570))
DROP VIEW IF EXISTS vw_ol_pass_status;
CREATE VIEW vw_ol_pass_status AS
SELECT vs.student_id, vs.registration_no, vs.student_name, vs.exam_year, vs.total_subjects, vs.passed_count,
       CASE
         WHEN vs.passed_count >= 4
           AND EXISTS(
             SELECT 1 FROM enrollments en JOIN results r2 ON r2.enrollment_id=en.enrollment_id
             JOIN subjects sub ON sub.subject_id=en.subject_id
             WHERE en.student_id=vs.student_id AND en.exam_year=vs.exam_year
               AND sub.code='0530' AND r2.grade IN ('A','B','C','D','E')
           )
           AND EXISTS(
             SELECT 1 FROM enrollments en JOIN results r2 ON r2.enrollment_id=en.enrollment_id
             JOIN subjects sub ON sub.subject_id=en.subject_id
             WHERE en.student_id=vs.student_id AND en.exam_year=vs.exam_year
               AND sub.code='0545' AND r2.grade IN ('A','B','C','D','E')
           )
           AND EXISTS(
             SELECT 1 FROM enrollments en JOIN results r2 ON r2.enrollment_id=en.enrollment_id
             JOIN subjects sub ON sub.subject_id=en.subject_id
             WHERE en.student_id=vs.student_id AND en.exam_year=vs.exam_year
               AND sub.code='0570' AND r2.grade IN ('A','B','C','D','E')
           )
         THEN 'PASS'
         ELSE 'FAIL'
       END AS ol_status
FROM vw_student_summary vs
WHERE vs.exam_year IS NOT NULL;

-- 5. Example queries you can run to test

-- a) List all students and their schools
SELECT s.registration_no, CONCAT(s.first_name,' ',s.last_name) AS name, sc.name AS school_name FROM students s LEFT JOIN schools sc ON s.school_id=sc.school_id;

-- b) View student result summary
SELECT * FROM vw_student_summary ORDER BY passed_count DESC;

-- c) View O/L pass status
SELECT * FROM vw_ol_pass_status;

-- d) Get subject pass rates
SELECT sub.code, sub.name,
SUM(CASE WHEN r.grade IN ('A','B','C','D','E') THEN 1 ELSE 0 END) AS pass_count,
COUNT(r.result_id) AS total_count,
ROUND(SUM(CASE WHEN r.grade IN ('A','B','C','D','E') THEN 1 ELSE 0 END)/COUNT(r.result_id)*100,2) AS pass_rate
FROM results r
JOIN enrollments e ON e.enrollment_id=r.enrollment_id
JOIN subjects sub ON sub.subject_id=e.subject_id
GROUP BY sub.subject_id, sub.code, sub.name ORDER BY pass_rate DESC;

-- 6. Stored procedure example: summary for a student by registration_no
DROP PROCEDURE IF EXISTS sp_student_summary;

CREATE PROCEDURE sp_student_summary(IN reg_no VARCHAR(50))
BEGIN
  SELECT s.registration_no,
         CONCAT(s.first_name,' ',s.last_name) AS student_name,
         e.exam_year,
         sub.code,
         sub.name AS subject_name,
         r.grade
  FROM students s
  JOIN enrollments e ON e.student_id = s.student_id
  JOIN subjects sub ON sub.subject_id = e.subject_id
  LEFT JOIN results r ON r.enrollment_id = e.enrollment_id
  WHERE s.registration_no = reg_no
  ORDER BY e.exam_year, sub.code;
END;


-- Example call:
CALL sp_student_summary('2025OL10001');





-- End of studentrecord.sql
