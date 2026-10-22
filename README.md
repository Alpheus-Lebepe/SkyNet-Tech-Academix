# SkyNet Tech Academix

SkyNet Tech Academix is a student registration and academic management system for storing student records, managing enrollment information, tracking academic status, and producing student-facing profile reports.

Repository: [Alpheus-Lebepe/SkyNet-Tech-Academix](https://github.com/Alpheus-Lebepe/SkyNet-Tech-Academix)

The repository contains a React frontend, a Node.js/Express backend, a MySQL database script, and a set of earlier PHP-oriented academic-management assets. The current frontend is designed around the React routes in `frontend/src/App.js`, while the backend contains both a working-looking inline API implementation in `backend/server.js` and separate route modules that are not currently mounted by that server. This README explains the intended architecture, the available workflows, the current source layout, setup, API behavior, and the changes required before production deployment.

## What the system does

The application supports two user experiences:

- **Administrators** can register students, search and filter student records, open student profiles, edit academic information, and delete records.
- **Students** can log in, view their dashboard and profile, review their course information, and generate a profile summary or registration confirmation slip for printing or PDF download.

The system stores personal and academic data such as names, identity numbers, student numbers, email addresses, dates of birth, courses, course summaries, enrollment dates, registration timestamps, passwords, and status values.

## Project status and implementation reality

The repository describes a modern full-stack application, but the current source contains two backend approaches at the same time:

1. `backend/server.js` creates a MySQL connection directly, exposes inline routes such as `/api/login` and `/api/students`, and listens on port `4000`.
2. `backend/routes/auth.js`, `backend/routes/students.js`, and `backend/middleware/auth.js` define a second Express-router/JWT approach. Those route modules are not mounted by the current `server.js`.

The frontend's `api.js` expects the server at `http://localhost:4000/api` and attaches a Bearer token from `localStorage`, but the inline routes in `server.js` primarily use `express-session` and do not issue the same JWT expected by the route modules. In addition, the separate route modules currently mix CommonJS and ES module conventions and contain unresolved variable references. For this reason, treat the repository as an active development project rather than a production-ready release.

The README documents the intended behavior and the current code as it exists. The known issues section identifies the changes needed to consolidate the backend into one consistent authentication and routing design.

## Main features

### Authentication and account access

The login interface supports two roles:

- Administrator
- Student

The login page asks for an email address, password, and role. The current inline API selects the `admins` table when the role is `admin` and the `students` table for other roles.

Administrators can register new administrator accounts from the login interface. The frontend checks that the name, email, password, and password confirmation are present before sending the registration request.

The login screen also includes a forgot-password interaction. It displays instructions to contact the IT Department for administrators or the course administrator for students. It is an instruction message, not an automated password-reset workflow.

### Administrator dashboard

The administrator dashboard is the central management screen. It loads student records and calculates status totals in the browser.

The dashboard provides cards for:

- Total Students
- Pending
- Awaiting Approval
- Active
- On Hold
- Suspended
- Graduated

Selecting a status card filters the table. The search field can match a student's:

- Full name
- Student number
- Email address

The table displays the student's name, student number, identity number, email, course, status, and a link to the detailed student view.

### Student registration

The administrator registration screen collects:

- Full name
- 13-digit identity number
- Student number
- Date of birth
- Email address
- Password
- Course
- Course summary
- Enrollment date
- Registration timestamp
- Status

The frontend automatically derives the date of birth from the first six digits of a 13-digit identity number in `YYMMDD` form. It also auto-fills a course summary when a course is selected.

The available course options are:

- System Development
- IT Security
- Networking
- AI & Data Science
- Full-Stack Dev

If no registration timestamp is entered, the frontend creates a current SQL-style timestamp before submitting the record.

The backend checks for duplicate email addresses, identity numbers, and student numbers before insertion.

### Student profile management

Administrators can open a student's profile using the student number in the route. The profile displays the stored personal and academic information and provides three actions:

- Edit Student
- Delete Student
- Back to Dashboard

The edit screen allows the administrator to update the student's details. The student number is read-only in the React edit component, while the backend route is capable of receiving a student-number field in the update payload.

Deleting a student requires a browser confirmation prompt. After deletion, the dashboard displays a success message using session storage.

### Student dashboard

After login, a student dashboard displays:

- Course
- Current academic status
- Enrollment date
- Registration timestamp
- Course overview

The dashboard also provides links to the student's profile and reports.

### Student profile

The student profile displays:

- Full name
- Identity number
- Student number
- Email
- Date of birth
- Course
- Course summary
- Enrollment date
- Academic status

The current component also renders the stored password. This is a serious security problem and should be removed immediately. Passwords should never be displayed, returned in ordinary profile responses, or stored in plaintext.

### Reports and document export

The student report screen has two tabs:

1. **Profile Summary** — a detailed personal and academic profile.
2. **Registration Confirmation Slip** — a formal registration-oriented summary containing the student, course, enrollment, registration, and status fields.

Both reports support:

- Browser printing.
- Client-side PDF generation.
- Returning to the student profile.

The frontend uses `html2canvas` to render report content and `jsPDF` to save the rendered result as a PDF. The generated filenames are `profile_summary.pdf` and `registration_slip.pdf`.

## Technology stack

### Frontend

- React 19
- React Router DOM
- Axios
- Create React App / `react-scripts`
- Boxicons
- `html2canvas`
- `jsPDF`
- React Testing Library

### Backend

- Node.js
- Express
- MySQL through `mysql2`
- `cors`
- `express-session`
- `body-parser`
- `jsonwebtoken`
- `bcryptjs`
- `dotenv`

### Database

- MySQL 8.x
- Database name: `student_db`
- SQL dump: `student_db.sql`

## Architecture

The intended request flow is:

```text
React browser application
        |
        | Axios / fetch requests to http://localhost:4000/api
        v
Node.js + Express API
        |
        | mysql2 queries
        v
MySQL database: student_db
```

The frontend uses `frontend/src/api.js` as its Axios client. The client uses:

```text
http://localhost:4000/api
```

It sends credentials and, when present, attaches a Bearer token stored under the `token` key in `localStorage`.

The current backend does not yet implement one fully consistent version of this architecture. The inline server uses session cookies, while the route modules describe JWT authentication. Consolidating this distinction is a required implementation task.

## Repository structure

```text
SkyNet-Tech-Academix/
├── README.md
├── student_db.sql
├── backend/
│   ├── .env
│   ├── db.js
│   ├── package.json
│   ├── package-lock.json
│   ├── server.js
│   ├── middleware/
│   │   └── auth.js
│   └── routes/
│       ├── auth.js
│       └── students.js
└── frontend/
    ├── .gitignore
    ├── package.json
    ├── package-lock.json
    ├── README.md
    ├── public/
    │   ├── favicon.ico
    │   ├── index.html
    │   ├── logo192.png
    │   ├── logo512.png
    │   ├── manifest.json
    │   └── robots.txt
    └── src/
        ├── App.js
        ├── App.test.js
        ├── api.js
        ├── index.js
        ├── index.css
        ├── logo.svg
        ├── reportWebVitals.js
        ├── setupTests.js
        ├── components/
        │   ├── AdminDashboard.jsx
        │   ├── AdminEditStudent.jsx
        │   ├── AdminRegisterStudent.jsx
        │   ├── AdminViewStudent.jsx
        │   ├── LoginPage.jsx
        │   ├── StudentDashboard.jsx
        │   ├── StudentProfile.jsx
        │   └── StudentProfileReport.jsx
        ├── context/
        │   └── AuthContext.js
        └── styles/
            ├── BkPic.jpg
            └── style.css
```

The repository also contains a committed `backend/node_modules` tree. It is normally better to remove committed dependencies from source control and rely on `package-lock.json` plus `npm install` or `npm ci`.

## Important files

| File | Purpose |
|---|---|
| `student_db.sql` | MySQL schema and demonstration records |
| `backend/server.js` | Current inline Express server, MySQL connection, session setup, and API routes |
| `backend/db.js` | Separate MySQL connection module used by the route-module approach |
| `backend/.env` | Backend environment variable file; it must not contain real committed secrets |
| `backend/middleware/auth.js` | JWT verification middleware for the modular route approach |
| `backend/routes/auth.js` | Modular login and administrator-registration routes |
| `backend/routes/students.js` | Modular student CRUD routes |
| `frontend/src/App.js` | React route map |
| `frontend/src/api.js` | Axios client and token interceptor |
| `frontend/src/context/AuthContext.js` | Local authentication state and token/admin storage helpers |
| `frontend/src/components/LoginPage.jsx` | Login, administrator registration, and forgot-password UI |
| `frontend/src/components/AdminDashboard.jsx` | Administrator student list and status dashboard |
| `frontend/src/components/AdminRegisterStudent.jsx` | Student registration form |
| `frontend/src/components/AdminViewStudent.jsx` | Student detail view and deletion action |
| `frontend/src/components/AdminEditStudent.jsx` | Student editing form |
| `frontend/src/components/StudentDashboard.jsx` | Student academic dashboard |
| `frontend/src/components/StudentProfile.jsx` | Student profile display |
| `frontend/src/components/StudentProfileReport.jsx` | Printable and downloadable reports |

## Database schema

The SQL dump creates a database model with two tables.

### `admins`

Stores administrator accounts:

- `id`
- `fullname`
- `password`
- `email`

The email field is unique.

### `students`

Stores student records:

- `fullname`
- `idNumber`
- `studentNo`
- `email`
- `dateofBirth`
- `password`
- `course`
- `courseSummary`
- `enrollmentDate`
- `status`
- `registrationTimestamp`

The SQL dump defines unique constraints for student number, email, and identity number.

The schema currently uses short `VARCHAR(45)` password fields and stores sample passwords as plain text. This design should be replaced with sufficiently sized password-hash columns before production use.

### Status values

The frontend and dashboard recognize these status values:

- `Pending`
- `Awaiting Approval`
- `Active`
- `On-hold`
- `Suspended`
- `Graduated`

The database uses a general `VARCHAR` status column rather than an enum or lookup table, so the backend should validate allowed values consistently.

## Installation prerequisites

Install the following software:

- Node.js 18 or later is recommended.
- npm.
- MySQL 8.x or a compatible MySQL server.
- A modern browser.
- Git.

Verify the tools:

```bash
node --version
npm --version
mysql --version
```

The frontend and backend have separate `package.json` files, so dependencies must be installed in both directories.

## Local setup

### 1. Clone the repository

```bash
git clone https://github.com/Alpheus-Lebepe/SkyNet-Tech-Academix.git
cd SkyNet-Tech-Academix
```

### 2. Create the MySQL database

Create the database expected by the application:

```sql
CREATE DATABASE student_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

Or run:

```bash
mysql -u root -p -e "CREATE DATABASE student_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

### 3. Import the schema and sample records

Import the SQL dump from the repository root:

```bash
mysql -u root -p student_db < student_db.sql
```

The dump drops and recreates the `admins` and `students` tables. Use this command only when you understand that existing development data will be replaced.

The SQL file includes sample administrator and student accounts. Treat those credentials as demonstration data only, and change or remove them before sharing the application with real users.

### 4. Configure backend environment variables

Create a local `backend/.env` file based on this structure:

```env
DB_HOST=127.0.0.1
DB_NAME=student_db
DB_USER=root
DB_PASS=your_mysql_password
JWT_SECRET=replace_with_a_long_random_secret
PORT=4000
```

Never commit real database passwords or JWT secrets. The repository currently contains a `.env` file and hard-coded database credentials in `server.js` and `db.js`. Rotate any exposed credentials and remove them from version control.

### 5. Install backend dependencies

```bash
cd backend
npm install
```

For reproducible installation after the dependency tree has been cleaned up:

```bash
npm ci
```

The current backend package defines:

```bash
npm start
```

which runs `node server.js`.

### 6. Install frontend dependencies

Open a second terminal from the repository root:

```bash
cd frontend
npm install
```

For a reproducible install:

```bash
npm ci
```

### 7. Start the application

Start the backend first:

```bash
cd backend
npm start
```

The intended API address is:

```text
http://localhost:4000
```

Start the frontend in a second terminal:

```bash
cd frontend
npm start
```

Create React App normally opens:

```text
http://localhost:3000
```

The frontend package declares a development proxy to `http://localhost:4000`, while `frontend/src/api.js` also uses an explicit API base URL. Keep these values aligned if the development ports change.

## Application routes

The React route map in `frontend/src/App.js` contains the following paths:

```text
/login
/admin_dashboard
/admin_view_student/:studentNo
/admin_edit_student/:studentNo
/admin_register_student
/student_dashboard
/student_profile
/student_profile_report
```

Unknown paths are redirected to `/login` by the wildcard route.

The UI also uses `/` in several logout links and redirects. If `/` is intended to be the login page, add an explicit route for it or consistently navigate to `/login`.

## Administrator process

### Sign in

1. Open `/login`.
2. Select `Administrator`.
3. Enter an administrator email and password.
4. Submit the form.
5. The login response supplies role and identity information.
6. The frontend stores basic identity values in `localStorage` and navigates to `/admin_dashboard`.

The intended secure version should return a short-lived access token or use a secure HTTP-only session cookie. It should never return or expose a plaintext password.

### Register a student

1. Open `/admin_register_student`.
2. Enter the student's personal and academic information.
3. Enter a 13-digit identity number.
4. Review the automatically derived date of birth.
5. Select a course.
6. Review the automatically populated course summary.
7. Enter enrollment and status information.
8. Submit the form.
9. The frontend sends `POST /api/students`.
10. The backend checks for duplicate email, identity number, and student number.
11. On success, the administrator is returned to the dashboard with a success message.

### Search and filter students

The dashboard first loads students from `/api/students`. Search is performed in the browser against the loaded list. Status cards also filter the in-memory collection.

This is convenient for a small dataset. For a large institution, move searching, pagination, sorting, and filtering to the backend so the browser does not need to load every student record.

### View a student

Selecting **View** navigates to:

```text
/admin_view_student/{studentNo}
```

The profile component requests the student record and displays the personal and academic fields. The current component fetches the complete student list and searches it in the browser rather than calling the more direct single-student endpoint. The backend does provide `GET /api/students/:studentNo` in the inline server and modular route file.

### Edit a student

1. Open a student profile.
2. Select **Edit Student**.
3. Change the required fields.
4. If the identity number changes to a valid 13-digit value, the frontend derives the date of birth.
5. If the course changes, the frontend replaces the course summary with the matching standard description.
6. Select **Save Changes**.
7. The frontend sends `PUT /api/students/{studentNo}`.
8. The backend checks for duplicate values belonging to another student.
9. The profile view is shown after a successful update.

### Delete a student

1. Open the student profile.
2. Select **Delete Student**.
3. Confirm the browser prompt.
4. The frontend sends `DELETE /api/students/{studentNo}`.
5. The student is deleted from MySQL.
6. The dashboard displays a success message.

Deletion is destructive. A production implementation should use audit logging, role-based authorization, CSRF protection, and preferably a soft-delete or archival strategy.

## Student process

### Sign in

1. Open `/login`.
2. Select `Student`.
3. Enter the student's email and password.
4. The login endpoint queries the `students` table.
5. On success, the frontend stores the student number, name, email, and role in `localStorage`.
6. The frontend navigates to `/student_dashboard`.

### View academic information

The student dashboard locates the student record using the stored email address and displays course, status, enrollment date, registration timestamp, and course summary.

The profile screen uses the stored student number to request `/api/students/:studentNo`.

### Generate a report

1. From the student dashboard, select **Reports**.
2. Choose **Profile Summary** or **Registration Slip**.
3. Select **Print** to open the browser print workflow.
4. Select **Download PDF** to render the report and save it as a PDF.
5. Select **Back** to return to the profile.

## API documentation

The application is not currently a fully standardized REST API, but the inline backend exposes these main endpoints.

### `POST /api/login`

Authenticates an administrator or student.

Request body:

```json
{
  "email": "user@example.com",
  "password": "password",
  "role": "admin"
}
```

The current server returns a response containing:

```json
{
  "message": "Loging in. please wait...",
  "role": "admin",
  "fullname": "Administrator Name",
  "email": "user@example.com",
  "studentNo": null
}
```

The current inline route sets `req.session.user` and `req.session.role`, but it does not return a JWT token. This conflicts with `frontend/src/api.js`, which expects a token in `localStorage` for protected requests.

### `POST /api/register_admin`

Registers an administrator.

Request body:

```json
{
  "fullname": "Admin Name",
  "email": "admin@example.com",
  "password": "password"
}
```

The endpoint checks whether the email already exists and inserts a new record when it does not.

### `GET /api/students`

Returns student records.

The inline server returns all columns from `students`. The modular route supports an optional search query:

```text
GET /api/students?q=networking
```

The modular route searches `fullname`, `studentNo`, and `email` and orders results by full name.

A production response should exclude password fields and return only the fields required by each screen.

### `GET /api/students/:studentNo`

Returns one student by student number.

Example:

```text
GET /api/students/13ha2304374
```

Returns `404` when the student does not exist.

### `POST /api/students`

Creates a student.

Example body:

```json
{
  "fullname": "Example Student",
  "idNumber": "0407255362086",
  "studentNo": "STU-1001",
  "dateofBirth": "2004-07-25",
  "email": "student@example.com",
  "password": "temporary-password",
  "course": "Networking",
  "courseSummary": "Course overview",
  "enrollmentDate": "2026-01-15",
  "registrationTimestamp": "2026-01-15 09:00:00",
  "status": "Pending"
}
```

The endpoint checks for duplicate email, identity number, and student number. The current implementation inserts the submitted password directly, so password hashing must be added before production use.

### `PUT /api/students/:studentNo`

Updates an existing student.

The route parameter identifies the original record. The request body contains the updated student fields. The backend checks for duplicate email, identity number, and student number while excluding the original record.

### `DELETE /api/students/:studentNo`

Deletes a student by student number.

The current inline route returns a `404` when no record is affected and a success object otherwise. Authorization must be enforced consistently before this endpoint is exposed.

### `POST /api/logout`

Destroys the Express session and returns:

```json
{
  "success": true
}
```

The frontend also clears browser storage during logout.

## Authentication design notes

There are currently two authentication designs in the repository.

### Session design in `server.js`

The inline server uses `express-session` with:

- A server-side session.
- A session cookie.
- `req.session.user`.
- `req.session.role`.

The current session secret is hard-coded as `skynet-secret`, and the cookie is configured with `secure: false`. This is acceptable only for local HTTP development and must be replaced in a deployed environment.

### JWT design in `routes/auth.js` and `middleware/auth.js`

The modular route files use `jsonwebtoken` and expect `JWT_SECRET` from the environment. The middleware reads:

```text
Authorization: Bearer <token>
```

and attaches an admin object to `req.admin` after verification.

The frontend Axios interceptor is compatible with this design because it reads `localStorage.getItem("token")` and sends a Bearer token. However, the current inline login route does not issue that token.

### Recommended consolidation

Choose one of the following designs and remove the other:

- **HTTP-only session design:** use secure, HTTP-only cookies and remove the JWT/localStorage flow.
- **JWT design:** issue JWTs from the active login route, store them safely, mount the modular routers, validate token expiry, and use a clear role claim.

For a browser-based student system, an HTTP-only secure session cookie is generally easier to protect against token theft than storing long-lived tokens in `localStorage`.

## Validation and data rules

The frontend applies several client-side rules:

- Required login role, email, and password.
- Required student name, identity number, student number, and email during registration.
- Identity-number input is limited to 13 characters.
- Email fields use HTML email validation.
- Student passwords use a minimum length in some forms.
- Course and status values are selected from fixed lists.
- Date of birth can be derived from an identity number.

Client-side validation improves usability but is not a security boundary. Every rule must also be implemented on the server.

The backend currently performs only limited validation. It checks required fields and duplicate values, but it does not consistently validate:

- Identity-number length and date validity.
- Email format.
- Password strength.
- Date formats.
- Allowed course values.
- Allowed status values.
- Field length limits.
- Whether the authenticated user is allowed to perform the requested operation.

## Security findings and production recommendations

The following changes are strongly recommended before the system handles real student records.

### Remove exposed secrets

The repository contains a committed `backend/.env` file and hard-coded MySQL credentials in `backend/server.js` and `backend/db.js`. Rotate those credentials immediately. Remove secret values from Git history if they were ever valid, and use a secret manager or deployment environment variables.

### Hash every password

The database dump contains plaintext-looking sample passwords, the inline login route compares passwords directly in SQL, and student creation/update routes accept and store passwords without hashing. Use:

```js
const hash = await bcrypt.hash(password, 12);
const valid = await bcrypt.compare(password, storedHash);
```

Never return password fields in API responses. Never render passwords in administrator or student profile screens.

### Fix authorization

The current inline `server.js` student routes do not consistently require authentication or an administrator role. The modular route file uses `auth`, but the current server does not mount it. Require authentication on all student-management endpoints and require an administrator role for create, update, and delete operations.

Students should only be able to read their own record. Do not rely solely on a student number or email stored in browser storage to authorize a request.

### Add CSRF protection

If the application uses cookies for sessions, add CSRF protection to all state-changing requests, including:

- Admin registration.
- Student creation.
- Student updates.
- Student deletion.
- Logout.

### Use secure cookies and secrets

Move the session secret to an environment variable. In production, use HTTPS and configure cookies with `secure: true`, `httpOnly: true`, and an appropriate `sameSite` value.

### Validate and limit input

Apply server-side validation for every field. Reject unexpected fields instead of inserting the entire request body with a broad `INSERT ... SET ?` operation. Validate allowed status and course values, identity-number format, email format, dates, and maximum lengths.

### Prevent sensitive data exposure

Do not return `SELECT *` from ordinary student endpoints. Explicitly select safe columns. The current frontend displays identity numbers, dates of birth, and passwords. Limit sensitive fields according to the user's role and operational need.

### Improve deletion and auditing

Consider replacing permanent deletion with a status such as archived or deleted. If deletion remains available, log who deleted the record, when it happened, and why. The current frontend confirmation prompt is useful for preventing accidental clicks but is not an audit mechanism.

### Configure CORS correctly

The server currently allows `http://localhost:3000`. Set the production frontend origin explicitly and do not use a wildcard origin with credentials.

## Known implementation issues

These issues are visible in the current repository and may prevent the application from running as intended without fixes.

### Backend module-system mismatch

`backend/package.json` sets `"type": "module"`, and `server.js` uses `import` syntax. The files in `backend/routes` and `backend/middleware` use `require()` and `module.exports`. Node cannot reliably use both styles without an explicit interoperability strategy. Convert all backend files to ES modules or remove `type: module` and convert the server consistently to CommonJS.

### Modular route files are not mounted

The current `server.js` does not import and mount `routes/auth.js` or `routes/students.js`. If the modular design is intended, mount routes explicitly, for example:

```js
app.use('/api/auth', authRoutes);
app.use('/api/students', studentRoutes);
```

Then align the frontend URLs and authentication behavior with those routes.

### Undefined variables in route modules

The modular `routes/auth.js` refers to `db.execute()` even though the imported connection variable is named `pool`. The modular `routes/students.js` contains a top-level `db.query(...)` call and uses `res` outside a request handler. These lines must be corrected or removed.

### Conflicting login endpoints

The frontend calls `/api/login` and `/api/register_admin`, while the modular auth router defines `/api/auth/login` and `/api/auth/register`. Choose one endpoint contract and update both client and server.

### Student route authorization is inconsistent

The modular route uses JWT middleware, but the inline server exposes student routes without the same middleware. This can create an authorization bypass if the inline endpoints are active.

### Student profile routes use inconsistent login paths

Some frontend components navigate to `/login`, while `StudentProfile.jsx` redirects to `/loginPage`. There is no `/loginPage` route in `App.js`; the defined route is `/login`. Standardize the route name.

### Logout navigation is inconsistent

Several components navigate to `/` after logout, while the React router redirects unknown paths to `/login`. This may work indirectly but should be made explicit and consistent.

### Test is still the Create React App placeholder

`frontend/src/App.test.js` searches for a `learn react` link. That test does not describe the current application and is expected to fail or provide no meaningful coverage after the UI changed. Replace it with tests for login validation, role redirects, dashboard rendering, form submission, and report controls.

### Dependency tree is committed

The repository contains `backend/node_modules`. Remove it from version control and add it to `.gitignore`. Keep `package-lock.json` so dependencies can be installed deterministically.

## Recommended development sequence

For the next development pass:

1. Rotate all exposed database and JWT/session secrets.
2. Remove `backend/node_modules` from Git and add ignore rules.
3. Choose session cookies or JWTs as the single authentication approach.
4. Convert the backend to one module system.
5. Move the database connection into one reusable module.
6. Mount one consistent set of API routers.
7. Add administrator and student authorization middleware.
8. Hash both administrator and student passwords.
9. Remove password fields from all API responses and React views.
10. Validate all server inputs and allowed status/course values.
11. Standardize `/login` and logout redirects.
12. Replace the placeholder frontend test with meaningful tests.
13. Add integration tests for authentication and student CRUD.
14. Add a deployment configuration with separate development and production environments.

## Testing checklist

Before considering the system ready for demonstration, verify:

- [ ] MySQL is running.
- [ ] `student_db` exists.
- [ ] `student_db.sql` imports successfully.
- [ ] Backend dependencies install without errors.
- [ ] Frontend dependencies install without errors.
- [ ] Backend starts on port `4000`.
- [ ] Frontend starts on port `3000`.
- [ ] CORS and the frontend proxy point to the same backend.
- [ ] Administrator login works.
- [ ] Student login works.
- [ ] Administrator registration works.
- [ ] Student registration rejects duplicate email, ID, and student numbers.
- [ ] Student registration derives date of birth correctly.
- [ ] Course selection fills the course summary.
- [ ] Dashboard status counts are correct.
- [ ] Dashboard search and filters work.
- [ ] Student profile loading works.
- [ ] Student editing works.
- [ ] Student deletion works only for authorized administrators.
- [ ] Students cannot access another student's record by changing the URL.
- [ ] Profile summary printing works.
- [ ] Registration-slip printing works.
- [ ] PDF downloads work.
- [ ] Logout invalidates the authenticated state.
- [ ] No password is returned or rendered.
- [ ] No secrets are committed.
- [ ] Backend integration tests cover all protected endpoints.

## License

No license is currently specified in this repository.

## References

1: https://github.com/Alpheus-Lebepe/SkyNet-Tech-Academix "SkyNet Tech Academix repository"

2: https://raw.githubusercontent.com/Alpheus-Lebepe/SkyNet-Tech-Academix/main/backend/server.js "SkyNet Tech Academix Express server"

3: https://raw.githubusercontent.com/Alpheus-Lebepe/SkyNet-Tech-Academix/main/backend/routes/students.js "SkyNet Tech Academix student routes"

4: https://raw.githubusercontent.com/Alpheus-Lebepe/SkyNet-Tech-Academix/main/student_db.sql "SkyNet Tech Academix database schema"

5: https://raw.githubusercontent.com/Alpheus-Lebepe/SkyNet-Tech-Academix/main/frontend/src/App.js "SkyNet Tech Academix React routes"


## Author

# Alpheus Lebepe
