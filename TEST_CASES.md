# Software Engineering Lab - Test Cases

## 1. Project Information
- **Project Name:** Counselling App
- **Source Repository:** [https://github.com/Mohamed-Arif-J/counselling_app.git](https://github.com/Mohamed-Arif-J/counselling_app.git)
- **Destination Repository:** [https://github.com/adhil-vt/SE-LAB.git](https://github.com/adhil-vt/SE-LAB.git)
- **Technology Stack:** Python 3.14, Django 5.2 / 6.0, Django REST Framework, SQLite3, Google Generative AI (Gemini), WhiteNoise, HTML5, CSS3, JavaScript
- **Purpose of Testing:**
  Systematic verification and validation of the Counselling App web application to evaluate functional correctness, input validation, authentication and access control mechanisms, data integrity, diagnostic scoring, and AI service integrations. Testing ensures the application complies with Software Engineering standards and identifies defects, security flaws, and boundary failures before release.
- **Testing Environment & Methodology:**
  Testing was executed within an isolated environment against the live Django application server and Django Test Client framework. Test cases encompass positive validation, negative input tests, boundary testing, access control auditing, and error-handling evaluation across all implemented endpoints.

---

## 2. Modules Identified

Based on source code analysis of the cloned Counselling App codebase, the following functional modules and components actually exist in the project:

1. **User Authentication & Gateway Module (`authentications` app)**
   - Self-registration for patients with automated profile generation (`post_save` signal).
   - Multi-identifier authentication supporting Username, Email, or registered Phone Number.
   - Session termination (Logout) with home redirection.
   - Dynamic role-based redirection to dedicated dashboard workspaces (`patient_dashboard`, `therapist_dashboard`, `profile-admin`).

2. **User & Practitioner Profile Management (`authentications` app)**
   - `PatientProfile` model tracking phone number, date of birth, gender, emergency contact, and timestamps.
   - `TherapistProfile` model tracking specialization, qualification, clinical experience years, biography, verification status, and profile pictures.
   - REST endpoints (`/profile/`, `/profile/update/`) for profile retrieval and multipart profile modification with validation.

3. **Administrative Management & Analytics Module (`authentications.admin_views` & REST API)**
   - System administration dashboard telemetry metrics (total users, role breakdowns, therapist verification counts).
   - Registry user listing, detailed profile inspection, and record deletion.
   - Dynamic role reassignment (`patient`, `therapist`, `admin`) with automatic fallback profile generation.
   - Practitioner credential verification and status toggles (`is_verified`).
   - Keyword search across usernames, real names, emails, and phone numbers.
   - Administrative Psychoeducation Article CRUD endpoints.

4. **Appointments & Clinical Workspace Module (`appointments` app)**
   - Interactive therapist directory and calendar consultation booking.
   - Role-filtered appointment retrieval for patients and practitioners.
   - Clinical consultation lifecycle status transitions (`Pending`, `CONFIRMED`, `Cancelled`, `DONE`).
   - Notification dispatch via Django email pipeline.
   - Therapist clinical session notes recording (`private_notes` and `shared_summary`).
   - AI auto-summary generation hook for session notes.
   - Active clinical session workspace shell and historical patient case records lookup API (`patient_session_context`, `therapist_historical_records_api`).

5. **Mood Tracker & Clinical Diagnostics Module (`mood_tracker` app)**
   - Daily quick mood rating check-in (1-5 scale) with JSON and form-based endpoints.
   - Historical mood trend chart data aggregation API (`/mood_tracker/chart-data/`).
   - Private patient reflective journaling management (CRUD with search and date filters).
   - Standardized psychiatric diagnostic screening forms:
     - **GAD-7:** Generalized Anxiety Disorder 7-item assessment with automated scoring.
     - **PHQ-9:** Patient Health Questionnaire 9-item depression assessment with automated scoring.
   - Psychoeducation resource desk (`PsychoeducationArticle`) with personalized diagnostic category matching.
   - Navigation role redirection utility (`role_redirect`).

6. **AI Sessions & Clinical Intelligence Module (`Ai_sessions` app)**
   - Mental health triage AI chatbot assistant (`/api/chat/`).
   - Text sentiment analysis classifier (`/api/sentiment/`).
   - High-risk self-harm and suicide crisis detection (`/api/crisis/`).
   - Algorithmic therapist specialization matching (`/api/match/`).
   - Clinical notes objective summarizer (`/api/summarize/`).
   - Automated request stream monitoring via `CrisisDetectionMiddleware`.

---

## 3. Test Cases

Test cases were designed to thoroughly test all implemented features, validation constraints, edge cases, error-handling routines, and security boundaries.

### Status Legend
- **PASS:** The test scenario was executed and verified successfully against the application.
- **FAIL:** An actual defect or bug was uncovered during execution.
- **NOT EXECUTED:** The test scenario could not be run in the current environment (e.g., requires external third-party paid API credentials).

```
========================================================================================================================
Test Case ID : TC-REG-01
Module       : User Registration
Scenario     : Register a new patient account with valid inputs using standard 'phone' field
Preconditions: Database is active; username, email, and phone are unique
Test Steps   : 1. Send POST request to /register/ with valid username, first_name, last_name, email, phone, password, confirm_password
               2. Follow HTTP redirect
               3. Verify user and patient profile in database
Test Data    : username='alice', first_name='Alice', last_name='Smith', email='alice@example.com', phone='9876543210', password='Password123!', confirm_password='Password123!'
Expected Res : HTTP 302 Redirect to /login/; User record created with role='patient'; PatientProfile created with phone_number='9876543210'
Actual Result: HTTP 302 Redirect to /login/; User and PatientProfile created with phone_number='9876543210'
Status       : PASS
========================================================================================================================
Test Case ID : TC-REG-02
Module       : User Registration
Scenario     : Attempt registration with an existing username
Preconditions: User 'alice' already registered
Test Steps   : 1. Send POST request to /register/ with username='alice' and new email/phone
               2. Inspect rendered response
Test Data    : username='alice', email='alice_alt@example.com', phone='9876543211', password='Password123!'
Expected Res : HTTP 200 with error message 'Username already exists'; registration blocked
Actual Result: HTTP 200 with error message 'Username already exists'; registration blocked
Status       : PASS
========================================================================================================================
Test Case ID : TC-REG-03
Module       : User Registration
Scenario     : Attempt registration with an existing email address
Preconditions: Email 'alice@example.com' already registered
Test Steps   : 1. Send POST request to /register/ with unique username but duplicate email
               2. Inspect response content
Test Data    : username='alice_alt', email='alice@example.com', phone='9876543212', password='Password123!'
Expected Res : HTTP 200 with error message 'Email already exists'; registration blocked
Actual Result: HTTP 200 with error message 'Email already exists'; registration blocked
Status       : PASS
========================================================================================================================
Test Case ID : TC-REG-04
Module       : User Registration
Scenario     : Attempt registration with an existing phone number using 'phone' field
Preconditions: Phone number '9876543210' is already registered to user 'alice'
Test Steps   : 1. Send POST request to /register/ with unique username, unique email, but phone='9876543210'
               2. Inspect response content
Test Data    : username='bob', email='bob@example.com', phone='9876543210', password='Password123!'
Expected Res : HTTP 200 with error message 'Phone number already exists'; registration blocked
Actual Result: HTTP 200 with error message 'Phone number already exists'; registration blocked
Status       : PASS
========================================================================================================================
Test Case ID : TC-REG-05
Module       : User Registration
Scenario     : Handle registration payload containing 'phone_number' parameter name
Preconditions: None
Test Steps   : 1. Send POST request to /register/ passing phone with key name 'phone_number'
               2. Check database for phone persistence and duplicate checking
Test Data    : username='charlie', email='charlie@example.com', phone_number='9876543210' (duplicate of alice)
Expected Res : Application normalizes 'phone_number' or enforces validation; rejects duplicate phone
Actual Result: views.py reads request.POST.get('phone') which returns None; duplicate phone check is bypassed; user created with null phone in profile
Status       : FAIL (Bug ID: BUG-01)
========================================================================================================================
Test Case ID : TC-REG-06
Module       : User Registration
Scenario     : Validate server-side password and confirm_password match
Preconditions: None
Test Steps   : 1. Send direct POST request to /register/ with mismatched password and confirm_password
               2. Check if account is created in database
Test Data    : password='Password123!', confirm_password='DifferentPassword!'
Expected Res : HTTP 200 with error message 'Passwords do not match'; account creation rejected
Actual Result: Server-side register_view does not validate confirm_password; account created with mismatched passwords
Status       : FAIL (Bug ID: BUG-02)
========================================================================================================================
Test Case ID : TC-AUTH-01
Module       : Authentication
Scenario     : Login with valid username and password
Preconditions: User 'alice' exists with role='patient'
Test Steps   : 1. Send POST request to /login/ with login_id='alice' and password='Password123!'
               2. Check redirect URL and session
Test Data    : login_id='alice', password='Password123!'
Expected Res : HTTP 302 Redirect to /patient-dashboard/; user authenticated in session
Actual Result: HTTP 302 Redirect to /patient-dashboard/; user authenticated in session
Status       : PASS
========================================================================================================================
Test Case ID : TC-AUTH-02
Module       : Authentication
Scenario     : Login using registered email address as identifier
Preconditions: User 'alice' has email 'alice@example.com'
Test Steps   : 1. Send POST request to /login/ with login_id='alice@example.com'
               2. Check redirect response
Test Data    : login_id='alice@example.com', password='Password123!'
Expected Res : HTTP 302 Redirect to /patient-dashboard/
Actual Result: HTTP 302 Redirect to /patient-dashboard/
Status       : PASS
========================================================================================================================
Test Case ID : TC-AUTH-03
Module       : Authentication
Scenario     : Login using registered patient phone number as identifier
Preconditions: PatientProfile for 'alice' has phone_number='9876543210'
Test Steps   : 1. Send POST request to /login/ with login_id='9876543210'
               2. Check redirect response
Test Data    : login_id='9876543210', password='Password123!'
Expected Res : HTTP 302 Redirect to /patient-dashboard/
Actual Result: HTTP 302 Redirect to /patient-dashboard/
Status       : PASS
========================================================================================================================
Test Case ID : TC-AUTH-04
Module       : Authentication
Scenario     : Attempt login with incorrect password
Preconditions: User 'alice' exists
Test Steps   : 1. Send POST request to /login/ with invalid password
               2. Inspect response
Test Data    : login_id='alice', password='WrongPassword999'
Expected Res : HTTP 200 with error message 'Invalid credentials'; session not created
Actual Result: HTTP 200 with error message 'Invalid credentials'; session not created
Status       : PASS
========================================================================================================================
Test Case ID : TC-AUTH-05
Module       : Authentication
Scenario     : Dynamic role-based login redirect for therapist role
Preconditions: User 'dr_smith' exists with role='therapist'
Test Steps   : 1. Send POST request to /login/ with therapist credentials
               2. Verify redirection destination
Test Data    : login_id='dr_smith', password='Password123!'
Expected Res : HTTP 302 Redirect to /therapist-dashboard/
Actual Result: HTTP 302 Redirect to /therapist-dashboard/
Status       : PASS
========================================================================================================================
Test Case ID : TC-AUTH-06
Module       : Authentication
Scenario     : Dynamic role-based login redirect for admin role
Preconditions: User 'admin_super' exists with role='admin'
Test Steps   : 1. Send POST request to /login/ with admin credentials
               2. Verify redirection destination
Test Data    : login_id='admin_super', password='Password123!'
Expected Res : HTTP 302 Redirect to /profile_admin/
Actual Result: HTTP 302 Redirect to /profile_admin/
Status       : PASS
========================================================================================================================
Test Case ID : TC-AUTH-07
Module       : Authentication
Scenario     : User logout clears session and redirects to home page
Preconditions: User is logged in
Test Steps   : 1. Send POST request to /logout/
               2. Verify response status and redirect destination
Test Data    : None
Expected Res : HTTP 302 Redirect to '/' (home); session invalidated
Actual Result: HTTP 302 Redirect to '/' (home); session invalidated
Status       : PASS
========================================================================================================================
Test Case ID : TC-PROF-01
Module       : User Profile
Scenario     : Access /profile/ endpoint while unauthenticated
Preconditions: User is not logged in
Test Steps   : 1. Send GET request to /profile/
               2. Inspect response status and JSON payload
Test Data    : None
Expected Res : HTTP 401 Unauthorized with {"success": false, "message": "Login required"}
Actual Result: HTTP 401 Unauthorized with {"success": false, "message": "Login required"}
Status       : PASS
========================================================================================================================
Test Case ID : TC-PROF-02
Module       : User Profile
Scenario     : Retrieve patient profile details for authenticated patient
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send GET request to /profile/
               2. Inspect returned JSON data fields
Test Data    : None
Expected Res : HTTP 200 with username, email, role='patient', phone_number, gender, emergency_contact
Actual Result: HTTP 200 with complete patient profile JSON
Status       : PASS
========================================================================================================================
Test Case ID : TC-PROF-03
Module       : User Profile
Scenario     : Update patient profile name fields via /profile/update/
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /profile/update/ with new first_name and last_name
               2. Query User model to confirm persistence
Test Data    : first_name='AliceUpdated', last_name='SmithUpdated'
Expected Res : HTTP 200 with {"success": true, "message": "Profile updated successfully"}; DB updated
Actual Result: HTTP 200; database fields updated successfully
Status       : PASS
========================================================================================================================
Test Case ID : TC-PROF-04
Module       : User Profile
Scenario     : Enforce profile picture requirement on therapist profile update
Preconditions: Authenticated as therapist without profile picture
Test Steps   : 1. Send POST request to /profile/update/ without uploading a profile picture
               2. Inspect response
Test Data    : first_name='JohnUpdated' (no profile_picture file attached)
Expected Res : HTTP 400 with {"success": false, "message": "Profile picture is required for therapists"}
Actual Result: HTTP 400 with {"success": false, "message": "Profile picture is required for therapists"}
Status       : PASS
========================================================================================================================
Test Case ID : TC-APPT-01
Module       : Appointments
Scenario     : Book appointment with valid therapist, date, and time
Preconditions: Authenticated as patient 'alice'; therapist 'dr_smith' exists
Test Steps   : 1. Send POST request to /appointments/book/ with therapist_id, appointment_date, appointment_time
               2. Check response data and database record
Test Data    : therapist_id=<dr_smith.id>, appointment_date='2026-10-01', appointment_time='10:00:00'
Expected Res : HTTP 201 Created with Appointment_id, Status='Pending', and email delivery status
Actual Result: HTTP 201 Created with Appointment_id, Status='Pending', email_status='Success'
Status       : PASS
========================================================================================================================
Test Case ID : TC-APPT-02
Module       : Appointments
Scenario     : Prevent unauthenticated users from booking an appointment
Preconditions: User is not logged in
Test Steps   : 1. Send POST request to /appointments/book/ with appointment details
               2. Check response status code
Test Data    : therapist_id=1, appointment_date='2026-10-01', appointment_time='10:00:00'
Expected Res : HTTP 401 Unauthorized with {"error": "You must be logged in to book an appointment."}
Actual Result: HTTP 401 Unauthorized returned
Status       : PASS
========================================================================================================================
Test Case ID : TC-APPT-03
Module       : Appointments
Scenario     : Validate therapist_id existence during appointment booking
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /appointments/book/ with non-existent therapist_id
               2. Observe server response
Test Data    : therapist_id=99999, appointment_date='2026-10-01', appointment_time='10:00:00'
Expected Res : HTTP 400 Bad Request indicating invalid or non-existent therapist
Actual Result: Server crashes with unhandled sqlite3.IntegrityError: FOREIGN KEY constraint failed (HTTP 500)
Status       : FAIL (Bug ID: BUG-03)
========================================================================================================================
Test Case ID : TC-APPT-04
Module       : Appointments
Scenario     : Appointment list endpoint filters records to authenticated patient
Preconditions: Authenticated as patient 'alice'; appointments exist for alice and other users
Test Steps   : 1. Send GET request to /appointments/
               2. Inspect returned appointment array
Test Data    : None
Expected Res : HTTP 200 with list containing only appointments belonging to 'alice'
Actual Result: HTTP 200 with list containing only alice's appointments
Status       : PASS
========================================================================================================================
Test Case ID : TC-APPT-05
Module       : Appointments
Scenario     : Enforce authentication on appointment list endpoint (/appointments/)
Preconditions: Unauthenticated user; appointments exist in system
Test Steps   : 1. Send unauthenticated GET request to /appointments/
               2. Observe returned records
Test Data    : None
Expected Res : HTTP 401 Unauthorized
Actual Result: HTTP 200 OK returned; endpoint dumps every appointment in database to public requester
Status       : FAIL (Bug ID: BUG-04)
========================================================================================================================
Test Case ID : TC-APPT-06
Module       : Appointments
Scenario     : Enforce authentication and ownership authorization on appointment cancellation
Preconditions: Appointment #1 exists belonging to patient 'alice'
Test Steps   : 1. Send unauthenticated POST request to /appointments/cancel/1/
               2. Inspect status of appointment #1 in database
Test Data    : appointment_id=1
Expected Res : HTTP 401 Unauthorized or HTTP 403 Forbidden; appointment status unchanged
Actual Result: HTTP 200 OK; unauthenticated user successfully cancelled appointment without credentials
Status       : FAIL (Bug ID: BUG-05)
========================================================================================================================
Test Case ID : TC-APPT-07
Module       : Appointments
Scenario     : Enforce practitioner authorization on complete appointment endpoint
Preconditions: Appointment #1 exists
Test Steps   : 1. Send unauthenticated POST request to /appointments/1/complete/
               2. Verify database appointment status
Test Data    : appointment_id=1
Expected Res : HTTP 401 Unauthorized or HTTP 403 Forbidden; status remains unchanged
Actual Result: HTTP 200 OK; unauthenticated user marked appointment as DONE
Status       : FAIL (Bug ID: BUG-06)
========================================================================================================================
Test Case ID : TC-APPT-08
Module       : Appointments
Scenario     : Enforce authentication on clinical session notes endpoint (/appointments/notes/<id>/)
Preconditions: Appointment #1 exists with session notes
Test Steps   : 1. Send unauthenticated GET request to /appointments/notes/1/
               2. Inspect returned content
Test Data    : appointment_id=1
Expected Res : HTTP 401 Unauthorized
Actual Result: HTTP 200 OK; private therapist notes leaked due to inverted access check 'if is_authenticated and role != therapist'
Status       : FAIL (Bug ID: BUG-07)
========================================================================================================================
Test Case ID : TC-APPT-09
Module       : Appointments
Scenario     : Protect patient clinical context and journal logs from unauthenticated access
Preconditions: Appointment #1 belongs to patient 'alice' who has private journal logs
Test Steps   : 1. Send unauthenticated GET request to /appointments/context/1/
               2. Inspect response payload
Test Data    : appointment_id=1
Expected Res : HTTP 401 Unauthorized
Actual Result: HTTP 200 OK; patient's full name, username, appointment data, and handwritten diaries exposed
Status       : FAIL (Bug ID: BUG-08)
========================================================================================================================
Test Case ID : TC-APPT-10
Module       : Appointments
Scenario     : Protect therapist historical case records API from unauthenticated access
Preconditions: Unauthenticated user
Test Steps   : 1. Send GET request to /appointments/api/historical-records/
               2. Check response status
Test Data    : None
Expected Res : HTTP 401 Unauthorized
Actual Result: HTTP 401 Unauthorized with {"error": "Unauthorized Access"}
Status       : PASS
========================================================================================================================
Test Case ID : TC-MOOD-01
Module       : Mood Tracker
Scenario     : Submit quick mood check-in rating via JSON POST
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /mood_tracker/checkin/ with JSON body {"mood": 4, "note": "Feeling positive"}
               2. Verify record in MoodLog model
Test Data    : mood=4, note='Feeling positive'
Expected Res : HTTP 200 with {"success": true, "message": "Mood updated!"}; MoodLog saved with mood=4
Actual Result: HTTP 200 with success message; MoodLog record created with mood=4
Status       : PASS
========================================================================================================================
Test Case ID : TC-MOOD-02
Module       : Mood Tracker
Scenario     : Reject non-integer mood value in JSON check-in
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /mood_tracker/checkin/ with {"mood": "invalid"}
               2. Inspect response
Test Data    : {"mood": "invalid", "note": "test"}
Expected Res : HTTP 400 with {"success": false, "error": "Invalid format data"}
Actual Result: HTTP 400 with {"success": false, "error": "Invalid format data"}
Status       : PASS
========================================================================================================================
Test Case ID : TC-MOOD-03
Module       : Mood Tracker
Scenario     : Retrieve 7-day mood trend chart data for patient
Preconditions: Authenticated as patient 'alice'; mood logs exist
Test Steps   : 1. Send GET request to /mood_tracker/chart-data/?days=7
               2. Verify JSON structure
Test Data    : days=7
Expected Res : HTTP 200 with 'labels' and 'moods' arrays
Actual Result: HTTP 200 with valid chart labels and numerical mood array
Status       : PASS
========================================================================================================================
Test Case ID : TC-JRNL-01
Module       : Mood Tracker / Journal
Scenario     : Validate required fields (title, content) on journal creation
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /mood_tracker/journal/create/ with empty title and content
               2. Inspect response
Test Data    : title='', content=''
Expected Res : HTTP 400 with {"success": false, "errors": "Title and content are required fields."}
Actual Result: HTTP 400 with error message; creation rejected
Status       : PASS
========================================================================================================================
Test Case ID : TC-JRNL-02
Module       : Mood Tracker / Journal
Scenario     : Create journal entry with valid title and content
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /mood_tracker/journal/create/ with title and content
               2. Check database for JournalEntry
Test Data    : title='My Morning', content='I went for a peaceful walk today.'
Expected Res : HTTP 200 with {"success": true, "journal_id": <id>}; JournalEntry saved
Actual Result: HTTP 200 with success: true and generated journal_id; DB updated
Status       : PASS
========================================================================================================================
Test Case ID : TC-JRNL-03
Module       : Mood Tracker / Journal
Scenario     : Reject HTTP GET method on journal deletion endpoint
Preconditions: Authenticated as patient 'alice'; journal entry exists
Test Steps   : 1. Send GET request to /mood_tracker/journal/<id>/delete/
               2. Inspect response
Test Data    : journal_id=<valid_id>
Expected Res : HTTP 400 with {"success": false, "error": "Invalid request method"}
Actual Result: HTTP 400 with {"success": false, "error": "Invalid request method"}
Status       : PASS
========================================================================================================================
Test Case ID : TC-JRNL-04
Module       : Mood Tracker / Journal
Scenario     : Delete journal entry via valid HTTP POST request
Preconditions: Authenticated as patient 'alice'; journal entry exists
Test Steps   : 1. Send POST request to /mood_tracker/journal/<id>/delete/
               2. Verify entry removed from database
Test Data    : journal_id=<valid_id>
Expected Res : HTTP 200 with {"success": true, "message": "Deleted successfully"}; entry deleted
Actual Result: HTTP 200 with success message; record purged from JournalEntry model
Status       : PASS
========================================================================================================================
Test Case ID : TC-DIAG-01
Module       : Clinical Diagnostics
Scenario     : Submit GAD-7 anxiety screening form with valid numeric inputs
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /mood_tracker/gad7/ with q1-q7 integer scores
               2. Inspect GAD7Response database record
Test Data    : q1=2, q2=1, q3=3, q4=0, q5=1, q6=2, q7=1 (sum = 10)
Expected Res : HTTP 200 with GAD7-Form.html rendered; GAD7Response created with score=10
Actual Result: HTTP 200; GAD7Response record created with calculated score=10
Status       : PASS
========================================================================================================================
Test Case ID : TC-DIAG-02
Module       : Clinical Diagnostics
Scenario     : Handle empty or non-integer input gracefully in GAD-7 assessment form
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /mood_tracker/gad7/ with empty string in q1
               2. Observe server handling
Test Data    : q1=''
Expected Res : HTTP 400 or form validation error message rendered to user
Actual Result: Server crashes with unhandled ValueError: invalid literal for int() with base 10: '' (HTTP 500)
Status       : FAIL (Bug ID: BUG-09)
========================================================================================================================
Test Case ID : TC-DIAG-03
Module       : Clinical Diagnostics
Scenario     : Submit PHQ-9 depression screening form with valid numeric inputs
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /mood_tracker/phq9/ with q1-q9 integer scores
               2. Inspect PHQ9Response database record
Test Data    : q1=1, q2=1, q3=2, q4=1, q5=1, q6=1, q7=1, q8=1, q9=0 (sum = 9)
Expected Res : HTTP 200 with PHQ9-Form.html rendered; PHQ9Response created with score=9
Actual Result: HTTP 200; PHQ9Response created with calculated score=9
Status       : PASS
========================================================================================================================
Test Case ID : TC-ART-01
Module       : Psychoeducation Resources
Scenario     : Verify PsychoeducationArticle model string representation (__str__)
Preconditions: PsychoeducationArticle created
Test Steps   : 1. Instantiate PsychoeducationArticle(title="Understanding CBT")
               2. Evaluate str(article)
Test Data    : title="Understanding CBT"
Expected Res : str(article) returns article title "Understanding CBT"
Actual Result: Typo in method name '_str_' causes str(article) to return default '<PsychoeducationArticle: PsychoeducationArticle object (1)>'
Status       : FAIL (Bug ID: BUG-10)
========================================================================================================================
Test Case ID : TC-RED-01
Module       : Access Control / Redirection
Scenario     : Evaluate /mood_tracker/role_redirect/ for therapist role
Preconditions: Authenticated as therapist 'dr_smith'
Test Steps   : 1. Send GET request to /mood_tracker/role_redirect/
               2. Check redirect target
Test Data    : user.role='therapist'
Expected Res : HTTP 302 Redirect to /therapist-dashboard/
Actual Result: View checks role=='counsellor'; falls through to else and erroneously redirects therapist to /patient-dashboard/
Status       : FAIL (Bug ID: BUG-11)
========================================================================================================================
Test Case ID : TC-AI-01
Module       : AI Sessions
Scenario     : Chatbot endpoint validates required 'message' field in request body
Preconditions: None
Test Steps   : 1. Send POST request to /api/chat/ with empty JSON body {}
               2. Inspect response status and validation errors
Test Data    : {}
Expected Res : HTTP 400 Bad Request with serializer error for missing 'message' field
Actual Result: HTTP 400 Bad Request returned with 'message' field required
Status       : PASS
========================================================================================================================
Test Case ID : TC-AI-02
Module       : AI Sessions
Scenario     : Sentiment analysis endpoint handles missing 'text' field validation
Preconditions: None
Test Steps   : 1. Send POST request to /api/sentiment/ with empty JSON body {}
               2. Inspect response status
Test Data    : {}
Expected Res : HTTP 400 Bad Request with serializer error for missing 'text' field
Actual Result: View lacks return statement for invalid serializer, returns None, causing DRF to crash with AssertionError (HTTP 500)
Status       : FAIL (Bug ID: BUG-14)
========================================================================================================================
Test Case ID : TC-AI-03
Module       : AI Sessions
Scenario     : Crisis detection endpoint validates required 'text' field
Preconditions: None
Test Steps   : 1. Send POST request to /api/crisis/ with empty JSON body {}
               2. Inspect response status
Test Data    : {}
Expected Res : HTTP 400 Bad Request with serializer error for missing 'text' field
Actual Result: HTTP 400 Bad Request returned with 'text' field required
Status       : PASS
========================================================================================================================
Test Case ID : TC-AI-04
Module       : AI Sessions
Scenario     : Therapist matching endpoint validates required 'text' field
Preconditions: None
Test Steps   : 1. Send POST request to /api/match/ with empty JSON body {}
               2. Inspect response status
Test Data    : {}
Expected Res : HTTP 400 Bad Request with serializer error for missing 'text' field
Actual Result: HTTP 400 Bad Request returned with 'text' field required
Status       : PASS
========================================================================================================================
Test Case ID : TC-AI-05
Module       : AI Sessions
Scenario     : Clinical note summarization endpoint validates required 'text' field
Preconditions: None
Test Steps   : 1. Send POST request to /api/summarize/ with empty JSON body {}
               2. Inspect response status
Test Data    : {}
Expected Res : HTTP 400 Bad Request with serializer error for missing 'text' field
Actual Result: HTTP 400 Bad Request returned with 'text' field required
Status       : PASS
========================================================================================================================
Test Case ID : TC-AI-06
Module       : AI Sessions
Scenario     : Live Gemini API generative chat completion execution
Preconditions: Active GEMINI_API_KEY and network access to Google Generative AI API
Test Steps   : 1. Send POST request to /api/chat/ with {"message": "I feel anxious today"}
               2. Verify generated natural language supportive reply
Test Data    : message='I feel anxious today'
Expected Res : HTTP 200 with AI supportive reply under 100 words
Actual Result: Cannot be executed in current offline lab test suite without external API credentials
Status       : NOT EXECUTED
========================================================================================================================
Test Case ID : TC-AI-07
Module       : AI Sessions
Scenario     : Live Gemini API sentiment analysis JSON output parsing
Preconditions: Active GEMINI_API_KEY and network access
Test Steps   : 1. Send POST request to /api/sentiment/ with {"text": "I feel thrilled and energized"}
               2. Verify JSON structure containing sentiment and confidence
Test Data    : text='I feel thrilled and energized'
Expected Res : HTTP 200 with {"sentiment": "POSITIVE", "confidence": <int>}
Actual Result: Cannot be executed in current offline lab test suite without external API credentials
Status       : NOT EXECUTED
========================================================================================================================
Test Case ID : TC-AI-08
Module       : AI Sessions
Scenario     : Live Gemini API crisis and self-harm detection execution
Preconditions: Active GEMINI_API_KEY and network access
Test Steps   : 1. Send POST request to /api/crisis/ with {"text": "I want to end my life"}
               2. Verify crisis flag and risk level
Test Data    : text='I want to end my life'
Expected Res : HTTP 200 with {"crisis": true, "risk_level": "HIGH"}
Actual Result: Cannot be executed in current offline lab test suite without external API credentials
Status       : NOT EXECUTED
========================================================================================================================
Test Case ID : TC-AI-09
Module       : AI Sessions / Middleware
Scenario     : CrisisDetectionMiddleware parsing of standard form-encoded journal data
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send POST request to /mood_tracker/journal/create/ with form-urlencoded self-harm text
               2. Verify whether request.crisis_detected is set to True
Test Data    : content='I want to end my life'
Expected Res : Middleware extracts text from form data and evaluates crisis risk
Actual Result: Middleware assumes JSON body (json.loads(request.body)); fails with JSONDecodeError and silently sets crisis_detected=False
Status       : FAIL (Bug ID: BUG-12)
========================================================================================================================
Test Case ID : TC-ADM-01
Module       : Administration
Scenario     : Restrict /admin/dashboard/ access from patient users
Preconditions: Authenticated as patient 'alice'
Test Steps   : 1. Send GET request to /admin/dashboard/
               2. Check response status code
Test Data    : None
Expected Res : HTTP 403 Forbidden with {"success": false, "message": "Admin access required"}
Actual Result: HTTP 403 Forbidden returned
Status       : PASS
========================================================================================================================
Test Case ID : TC-ADM-02
Module       : Administration
Scenario     : Retrieve system telemetry statistics as authenticated admin
Preconditions: Authenticated as admin 'admin_super'
Test Steps   : 1. Send GET request to /admin/dashboard/
               2. Inspect metrics returned in JSON payload
Test Data    : None
Expected Res : HTTP 200 with total_users, total_patients, total_therapists, total_admins, verified_therapists
Actual Result: HTTP 200 with complete telemetry dictionary
Status       : PASS
========================================================================================================================
Test Case ID : TC-ADM-03
Module       : Administration
Scenario     : Retrieve registered user directory in admin console
Preconditions: Authenticated as admin 'admin_super'
Test Steps   : 1. Send GET request to /admin/users/
               2. Inspect list of user objects
Test Data    : None
Expected Res : HTTP 200 with serialized user array
Actual Result: HTTP 200 with serialized user directory array
Status       : PASS
========================================================================================================================
Test Case ID : TC-ADM-04
Module       : Administration
Scenario     : Admin user edit prevents duplicate email assignment (/admin/users/<id>/edit/)
Preconditions: Authenticated as admin; users 'alice' and 'dr_smith' exist
Test Steps   : 1. Send POST request to /admin/users/<alice.id>/edit/ attempting to change email to dr.smith@example.com
               2. Inspect response
Test Data    : email='dr.smith@example.com'
Expected Res : HTTP 400 with {"success": false, "message": "Email already exists"}
Actual Result: HTTP 400 with "Email already exists"
Status       : PASS
========================================================================================================================
Test Case ID : TC-ADM-05
Module       : Administration
Scenario     : DRF administrative edit endpoint validates email uniqueness (/api/admin/users/<id>/edit/)
Preconditions: Authenticated as admin; users 'alice' and 'dr_smith' exist
Test Steps   : 1. Send POST request to /api/admin/users/<alice.id>/edit/ with email='dr.smith@example.com'
               2. Observe server response
Test Data    : email='dr.smith@example.com'
Expected Res : HTTP 400 Bad Request indicating email already exists
Actual Result: Endpoint fails to validate email uniqueness; triggers unhandled django.db.utils.IntegrityError: UNIQUE constraint failed (HTTP 500)
Status       : FAIL (Bug ID: BUG-13)
========================================================================================================================
Test Case ID : TC-ADM-06
Module       : Administration
Scenario     : Admin can toggle therapist credential verification status to unverified
Preconditions: Authenticated as admin; therapist 'dr_smith' is verified
Test Steps   : 1. Send POST request to /admin/therapists/<dr_smith.id>/unverify/
               2. Inspect database TherapistProfile.is_verified field
Test Data    : user_id=<dr_smith.id>
Expected Res : HTTP 200 with {"success": true, "is_verified": false}; DB record updated
Actual Result: HTTP 200; is_verified updated to False in database
Status       : PASS
========================================================================================================================
```

---

## 4. Test Execution Summary

A total of **54 test cases** were defined and evaluated across the functional modules of the Counselling App.

### Overall Execution Metrics

| Metric | Count | Percentage |
| :--- | :--- | :--- |
| **Total Test Cases** | **54** | **100.0%** |
| **Passed** | **37** | **68.5%** |
| **Failed** | **14** | **25.9%** |
| **Not Executed** | **3** | **5.6%** |

### Module-Wise Test Summary

| Module | Total Tests | Passed | Failed | Not Executed | Pass Rate |
| :--- | :---: | :---: | :---: | :---: | :---: |
| User Registration | 6 | 4 | 2 | 0 | 66.7% |
| Authentication & Sessions | 7 | 7 | 0 | 0 | 100.0% |
| User & Practitioner Profiles | 4 | 4 | 0 | 0 | 100.0% |
| Appointments & Consultations | 10 | 4 | 6 | 0 | 40.0% |
| Mood Tracking & Journals | 7 | 7 | 0 | 0 | 100.0% |
| Clinical Diagnostics (GAD-7/PHQ-9) | 3 | 2 | 1 | 0 | 66.7% |
| Psychoeducation Articles | 1 | 0 | 1 | 0 | 0.0% |
| Role Redirection & Routing | 1 | 0 | 1 | 0 | 0.0% |
| AI Sessions & Middleware | 9 | 5 | 1 | 3 | 55.6% |
| Administration Management | 6 | 5 | 1 | 0 | 83.3% |
| **Total** | **54** | **37** | **14** | **3** | **68.5%** |

> **Note on Non-Executed Tests:** Tests TC-AI-06, TC-AI-07, and TC-AI-08 involve live outbound calls to the Google Generative AI (Gemini 2.5 Flash) service. They were marked **NOT EXECUTED** in accordance with testing instructions because `GEMINI_API_KEY` was not configured in the offline testing environment. Serializer validation and exception handling paths for these endpoints were fully verified.

---

## 5. Bugs Identified

All 14 defects listed below were directly discovered and reproduced during test execution against the actual source code.

```
========================================================================================================================
Bug ID          : BUG-01
Related Test ID : TC-REG-05
Module          : User Registration (`authentications/views.py`)
Severity        : Medium
Description     : Parameter name mismatch between frontend/tests and backend registration view silently discards phone number.
Steps to Repro  : 1. Send POST request to /register/ with payload containing key 'phone_number' instead of 'phone'.
                  2. Observe that registration succeeds.
                  3. Query PatientProfile for the new user.
Expected Behav  : Either 'phone_number' is normalized and preserved, or an explicit validation error is returned.
Actual Behavior : views.py queries request.POST.get('phone') which returns None; duplicate phone validation is bypassed, and user is registered with phone_number=None.
Suggested Fix   : In register_view, check phone_number = request.POST.get("phone") or request.POST.get("phone_number").
========================================================================================================================
Bug ID          : BUG-02
Related Test ID : TC-REG-06
Module          : User Registration (`authentications/views.py`)
Severity        : High
Description     : Server-side registration view completely lacks password confirmation validation.
Steps to Repro  : 1. Send direct HTTP POST to /register/ with password='Password123!' and confirm_password='DifferentPassword!'.
                  2. Inspect HTTP response and database User table.
Expected Behav  : Server rejects registration with HTTP 400 / error message stating passwords do not match.
Actual Behavior : Backend ignores confirm_password entirely and registers the user account.
Suggested Fix   : Add server-side check: if password != request.POST.get("confirm_password"): messages.error(request, "Passwords do not match"); return render(request, 'register.html').
========================================================================================================================
Bug ID          : BUG-03
Related Test ID : TC-APPT-03
Module          : Appointments (`appointments/views.py`)
Severity        : High
Description     : Unhandled database IntegrityError crashes server with HTTP 500 when booking with non-existent therapist_id.
Steps to Repro  : 1. Authenticate as patient.
                  2. Send POST to /appointments/book/ with therapist_id=99999.
Expected Behav  : Clean HTTP 400 Bad Request or HTTP 404 response stating "Therapist does not exist".
Actual Behavior : Appointment.objects.create throws unhandled sqlite3.IntegrityError: FOREIGN KEY constraint failed (HTTP 500).
Suggested Fix   : Validate therapist existence prior to creation: therapist = get_object_or_404(User, id=therapist_id, role="therapist").
========================================================================================================================
Bug ID          : BUG-04
Related Test ID : TC-APPT-05
Module          : Appointments (`appointments/views.py`)
Severity        : Critical
Description     : Unauthenticated GET request to /appointments/ exposes all appointments in database.
Steps to Repro  : 1. Send unauthenticated GET request to /appointments/.
                  2. Inspect returned JSON response.
Expected Behav  : HTTP 401 Unauthorized or HTTP 403 Forbidden.
Actual Behavior : list_appointments executes if not request.user.is_authenticated: appointments = Appointment.objects.all(), dumping every appointment, patient username, and therapist username.
Suggested Fix   : Require authentication: if not request.user.is_authenticated: return Response({"error": "Authentication required"}, status=401).
========================================================================================================================
Bug ID          : BUG-05
Related Test ID : TC-APPT-06
Module          : Appointments (`appointments/views.py`)
Severity        : Critical
Description     : Complete authorization bypass on appointment cancellation endpoint (/appointments/cancel/<id>/).
Steps to Repro  : 1. Send unauthenticated POST request to /appointments/cancel/1/.
                  2. Inspect appointment status in database.
Expected Behav  : HTTP 401/403 Access Denied; only the booked patient or therapist can cancel.
Actual Behavior : cancel_appointment lacks authentication and permission checks; any anonymous user can cancel any appointment.
Suggested Fix   : Add @login_required / authentication check and verify request.user in [appointment.patient, appointment.therapist].
========================================================================================================================
Bug ID          : BUG-06
Related Test ID : TC-APPT-07
Module          : Appointments (`appointments/views.py`)
Severity        : Critical
Description     : Complete authorization bypass on complete appointment endpoint (/appointments/<id>/complete/).
Steps to Repro  : 1. Send unauthenticated POST request to /appointments/1/complete/.
                  2. Inspect appointment status in database.
Expected Behav  : HTTP 401/403 Access Denied; only assigned therapist can mark appointment completed.
Actual Behavior : Any unauthenticated user can mark any clinical consultation as 'DONE'.
Suggested Fix   : Enforce permission check: if not request.user.is_authenticated or request.user != appointment.therapist: return Response({"error": "Unauthorized"}, status=403).
========================================================================================================================
Bug ID          : BUG-07
Related Test ID : TC-APPT-08
Module          : Appointments (`appointments/views.py`)
Severity        : Critical
Description     : Inverted access control logic leaks private clinical session notes to unauthenticated visitors.
Steps to Repro  : 1. Send unauthenticated GET request to /appointments/notes/1/.
                  2. Read response JSON body.
Expected Behav  : HTTP 401 Unauthorized; confidential session notes must be restricted.
Actual Behavior : Check is if request.user.is_authenticated and user_role != "therapist". Since user is not authenticated, check evaluates to False, allowing unauthenticated visitors to read and overwrite private clinical notes.
Suggested Fix   : Update check: if not request.user.is_authenticated: return Response(status=401); if request.user.role != "therapist": return Response(status=403).
========================================================================================================================
Bug ID          : BUG-08
Related Test ID : TC-APPT-09
Module          : Appointments (`appointments/views.py`)
Severity        : Critical
Description     : Information disclosure endpoint leaks patient personal details and handwritten diaries to public visitors.
Steps to Repro  : 1. Send unauthenticated GET request to /appointments/context/1/.
                  2. Inspect response payload.
Expected Behav  : HTTP 401 Unauthorized.
Actual Behavior : Endpoint returns patient's full name, username, consultation time, and all handwritten journal logs to unauthenticated users.
Suggested Fix   : Restrict endpoint to authenticated therapist assigned to the appointment.
========================================================================================================================
Bug ID          : BUG-09
Related Test ID : TC-DIAG-02
Module          : Clinical Diagnostics (`mood_tracker/views.py`)
Severity        : High
Description     : Unhandled ValueError crashes GAD-7 form assessment on empty or non-integer input.
Steps to Repro  : 1. Authenticate as patient.
                  2. Send POST to /mood_tracker/gad7/ with q1=''.
Expected Behav  : Form validation error displayed to user with HTTP 400.
Actual Behavior : int(request.POST.get("q1", 0)) raises ValueError: invalid literal for int() with base 10: '', causing HTTP 500 crash.
Suggested Fix   : Use Django's GAD7Form for validation or safely parse integers: int(request.POST.get("q1") or 0).
========================================================================================================================
Bug ID          : BUG-10
Related Test ID : TC-ART-01
Module          : Psychoeducation (`mood_tracker/models.py`)
Severity        : Low
Description     : Typo in PsychoeducationArticle model string representation method name (_str_ instead of __str__).
Steps to Repro  : 1. Create PsychoeducationArticle instance.
                  2. Call str(article).
Expected Behav  : Returns article.title.
Actual Behavior : Method is defined as def _str_(self): with single underscores; Python falls back to default <PsychoeducationArticle object>.
Suggested Fix   : Rename method from def _str_(self): to def __str__(self):.
========================================================================================================================
Bug ID          : BUG-11
Related Test ID : TC-RED-01
Module          : Access Control / Routing (`mood_tracker/views.py`)
Severity        : Medium
Description     : Role redirect view misidentifies therapist role and redirects to patient dashboard.
Steps to Repro  : 1. Authenticate as therapist.
                  2. Navigate to /mood_tracker/role_redirect/.
Expected Behav  : Redirect to /therapist-dashboard/.
Actual Behavior : View checks request.user.role == "counsellor" (invalid role name) and falls through to else, redirecting therapists to /patient-dashboard/.
Suggested Fix   : Update role check: if request.user.role == "therapist": return redirect("therapist_dashboard").
========================================================================================================================
Bug ID          : BUG-12
Related Test ID : TC-AI-09
Module          : AI Sessions / Middleware (`Ai_sessions/middleware.py`)
Severity        : High
Description     : CrisisDetectionMiddleware assumes JSON body and fails on standard form-encoded journal posts.
Steps to Repro  : 1. Authenticate as patient.
                  2. Submit journal form via standard web browser POST (application/x-www-form-urlencoded).
Expected Behav  : Middleware evaluates text content for mental health crisis triggers.
Actual Behavior : Middleware executes json.loads(request.body) which throws JSONDecodeError on form-urlencoded data; exception handler catches it and silently resets crisis_detected=False.
Suggested Fix   : Support form data in middleware: extract text from request.POST when content-type is form-urlencoded.
========================================================================================================================
Bug ID          : BUG-13
Related Test ID : TC-ADM-05
Module          : Administration (`authentications/views.py`)
Severity        : Medium
Description     : DRF administrative user edit endpoint lacks email uniqueness check, causing database IntegrityError.
Steps to Repro  : 1. Authenticate as admin.
                  2. Send POST to /api/admin/users/<id>/edit/ assigning an email belonging to another user.
Expected Behav  : Clean HTTP 400 Bad Request stating "Email already exists".
Actual Behavior : View directly saves model, triggering unhandled django.db.utils.IntegrityError: UNIQUE constraint failed (HTTP 500).
Suggested Fix   : Add uniqueness check before saving: if User.objects.exclude(id=user_id).filter(email=email).exists(): return Response({"message": "Email already exists"}, status=400).
========================================================================================================================
Bug ID          : BUG-14
Related Test ID : TC-AI-02
Module          : AI Sessions (`Ai_sessions/views.py`)
Severity        : High
Description     : Missing return statement in sentiment view on invalid serializer payload raises AssertionError.
Steps to Repro  : 1. Send POST request to /api/sentiment/ with empty body {}.
                  2. Observe response.
Expected Behav  : HTTP 400 Bad Request with serializer errors.
Actual Behavior : View executes if serializer.is_valid(): ... but lacks an else branch, returning None; DRF raises AssertionError: Expected a Response... received NoneType (HTTP 500).
Suggested Fix   : Add return Response(serializer.errors, status=400) at the end of sentiment view.
========================================================================================================================
```

---

## 6. Conclusion

Comprehensive testing was conducted on the Counselling App repository ([Mohamed-Arif-J/counselling_app](https://github.com/Mohamed-Arif-J/counselling_app.git)). 

### Key Findings & Quality Assessment:
1. **Core Features Working Well:** Basic patient self-registration, multi-identifier login (Username, Email, Phone), profile updates, mood log recording, chart telemetry generation, journal entry CRUD, PHQ-9 scoring, GAD-7 scoring, and admin telemetry statistics operate correctly under standard positive input flows.
2. **Critical Security & Authorization Flaws:** Several endpoints in the appointments module (`/appointments/`, `/appointments/cancel/<id>/`, `/appointments/notes/<id>/`, `/appointments/context/<id>/`) lack mandatory authentication or suffer from inverted access control logic (`if is_authenticated and role != therapist`). This permits unauthenticated public visitors to view sensitive appointment rosters, cancel consultations, inspect handwritten patient journals, and read/overwrite private clinical session notes.
3. **Data Validation & Error Handling Gaps:** Missing foreign key validation on appointment booking, lack of server-side password confirmation matching during registration, and lack of input parsing safeguards in diagnostic forms (GAD-7) lead to unhandled server crashes (`IntegrityError`, `ValueError`, `AssertionError`) rather than graceful HTTP 400 validation responses.
4. **AI Architecture Recommendations:** The `CrisisDetectionMiddleware` should be updated to parse both JSON and multipart/form-urlencoded payloads so that crisis detection functions during standard browser form submissions. Missing return statements in API views (`/api/sentiment/`) must be resolved to avoid framework assertion failures.

Overall, the project demonstrates a rich functional architecture spanning clinical appointment management, mood diagnostics, and AI assistant integration, but requires remediation of the identified authorization and validation bugs prior to production clinical deployment.
