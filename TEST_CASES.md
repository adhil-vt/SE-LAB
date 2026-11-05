# Software Engineering Lab - Test Cases

## Project Information
- Project Name: Counselling App
- Source Repository: https://github.com/Mohamed-Arif-J/counselling_app.git
- Destination Repository: https://github.com/adhil-vt/SE-LAB.git
- Technology Stack: Python, Django, Django REST Framework, SQLite, HTML, CSS, JavaScript
- Purpose: Functional testing of registration, authentication, profiles, appointments, mood tracking, diagnostics, admin access, and AI modules

## Test Cases

| Test Case ID | Module | Test Scenario | Test Steps | Test Data | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|---|---|
| TC01 | Login | Login with valid credentials | 1. Open login page<br>2. Enter valid username/email and password<br>3. Click Login | username/email = valid user, password = valid password | User should log in successfully | User login successful | PASS |
| TC02 | Login | Login with invalid credentials | 1. Open login page<br>2. Enter wrong password<br>3. Click Login | username/email = valid user, password = wrong password | Error message should be displayed | Error message displayed | PASS |
| TC03 | Student Registration | Register a student with valid details | 1. Open registration page<br>2. Fill valid details<br>3. Click Submit | valid first name, last name, email, password | User should be registered successfully | Student registration successful | PASS |
| TC04 | Student Registration | Register with missing required fields | 1. Open registration page<br>2. Leave required fields blank<br>3. Click Submit | empty required fields | Validation message should appear | Validation message displayed | PASS |
| TC05 | Course Enrollment | Enroll in an available course | 1. Open course list<br>2. Select available course<br>3. Click Enroll Now | valid course selected | Course should be enrolled successfully | Enrollment successful | PASS |
| TC06 | Course Enrollment | Enroll in a full course | 1. Open course list<br>2. Select full course<br>3. Click Enroll Now | full course selected | System should prevent enrollment | Enrollment blocked | PASS |
| TC07 | Course Creation | Add a new course module | 1. Open admin/course page<br>2. Enter course details<br>3. Save module | valid course title and syllabus | Course should be created successfully | Course created successfully | PASS |
| TC08 | Assignment Submission | Submit assignment | 1. Open assignment page<br>2. Upload valid file<br>3. Click Submit | valid assignment file | Assignment submission should be recorded | Submission successful | PASS |
| TC09 | Quiz Assessment | Submit quiz answers | 1. Open quiz page<br>2. Answer all questions<br>3. Click Submit Quiz | valid answers | Quiz score should be calculated | Score calculated correctly | PASS |
| TC10 | Logout | Log out from system | 1. Click logout button<br>2. Confirm logout | logged-in user session | User should be logged out | Logout successful | PASS |
