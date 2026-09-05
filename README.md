# EduAuth Registry

**A secure platform for issuing, managing, and verifying academic certificates.**

![Java 17](https://img.shields.io/badge/Java-17-007396?logo=java) ![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.3.2-6DB33F?logo=springboot) ![React](https://img.shields.io/badge/React-18-61DAFB?logo=react) ![MySQL](https://img.shields.io/badge/MySQL-8-4479A1?logo=mysql) ![License](https://img.shields.io/badge/License-MIT-green)

---

> ⚡ **Quick Start Heads-Up**
> - **Ports:** Backend on `8080`, Frontend on `5173`
> - **Start Order:** MySQL first → Backend second → Frontend third
> - **Required Config:** DB password, 256-bit JWT secret, and Gmail App Password (in `application.properties`)

---

EduAuth Registry is a full-stack web application that enables universities to issue tamper-evident digital academic certificates, students to manage and share their credentials, and employers or institutions to verify authenticity in real time. The system supports role-based access across four user types with JWT-based authentication and OTP email verification.

## Features

- Role-based access control (Student, University, Verifier, Admin)
- JWT authentication with OTP email verification
- Certificate issuance with unique serial numbers and checksum validation
- PDF certificate generation with embedded QR codes (iText 7 + ZXing)
- Public/private visibility toggle per certificate
- Share-link verification with encrypted date-of-birth tokens
- Verification logging with full audit history and CSV export
- Admin controls for user approval, suspension, and certificate revocation

## Tech Stack

| Layer        | Technology                           |
|--------------|--------------------------------------|
| Backend      | Java 17, Spring Boot 3.3.2, Spring Security |
| Frontend     | React 18, Vite, Tailwind CSS         |
| Database     | MySQL 8                              |
| Security     | JWT (jjwt 0.12.6), BCrypt            |
| Build Tool   | Maven 3.9.6 (bundled), npm           |
| PDF          | iText 7                              |
| QR Code      | ZXing 3.5.3 (backend), qrcode.react (frontend) |
| Email        | Spring Mail (Gmail SMTP)             |

## User Roles

| Role        | Capabilities                                                   |
|-------------|----------------------------------------------------------------|
| Student     | View and download own certificates, toggle visibility, share links |
| University  | Issue certificates to students, view issued certificates       |
| Verifier    | Verify certificates by serial, view and export verification history |
| Admin       | Manage all users and certificates, revoke/restore, approve accounts |

## Getting Started

> Start everything in this order: MySQL → Backend → Frontend
> The backend will not work without MySQL running first.

### Prerequisites

- Java 17+
- Maven 3.9+ — if not installed, the project includes a bundled Maven at `backend/apache-maven-3.9.6` that you can use directly
- MySQL 8+
- Node.js 18+

### Clone the Repository

```bash
git clone https://github.com/litch07/eduauth-registry-maven.git
cd eduauth-registry-maven
```

### Database Setup

#### Step 1: Open MySQL and create the database
You can run this from your terminal:
```sql
CREATE DATABASE eduauth_registry;
```
Or directly from the command line:
```bash
mysql -u root -p -e "CREATE DATABASE eduauth_registry;"
```
*(Alternatively, you can use a GUI like MySQL Workbench or phpMyAdmin if you prefer.)*

#### Step 2: Import schema
```bash
mysql -u root -p eduauth_registry < db/schema.sql
```

#### Step 3: Import seed data (test accounts)
```bash
mysql -u root -p eduauth_registry < db/seed.sql
```

#### Step 4: Verify it worked
```bash
mysql -u root -p -e "USE eduauth_registry; SHOW TABLES;"
```
You should see about 15 tables listed.

> **Note:** If MySQL is not installed, download it from https://dev.mysql.com/downloads/

### Backend Setup

#### Step 1: Navigate to backend folder
```bash
cd backend
```

#### Step 2: Open application.properties and fill in these values
Open `src/main/resources/application.properties` (or copy from `application.properties.example` if creating fresh) and configure the following:

- **`spring.datasource.password`**
  - Set this to your MySQL root password (or whatever user you created).

- **`jwt.secret`**
  - This must be a 256-bit Base64 encoded secret key.
  - To generate one:
    - **Option A (Linux/Mac terminal):**
      ```bash
      openssl rand -base64 32
      ```
    - **Option B (Windows PowerShell):**
      ```powershell
      [Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Max 256 }))
      ```
    - **Option C (online):**
      Go to https://generate-secret.vercel.app/32 and copy the result.
  - Paste the output as the value of `jwt.secret`.

- **`spring.mail.username`**
  - Your Gmail address (e.g. `yourname@gmail.com`).

- **`spring.mail.password`**
  - This is **NOT** your Gmail login password.
  - It is a Google App Password. To get one:
    1. Go to https://myaccount.google.com
    2. Click **Security**
    3. Under "How you sign in to Google", click **2-Step Verification** (enable it if not already on)
    4. Scroll to the bottom, click **App passwords**
    5. Select app: **Mail**, Select device: **Other**, type `"EduAuth"`
    6. Click **Generate** — copy the 16-character password shown
    7. Paste that password as the value of `spring.mail.password`

#### Step 3: Run the backend
```bash
mvn spring-boot:run
```
*(If `mvn` is not installed, use the bundled Maven:)*
```bash
./apache-maven-3.9.6/bin/mvn spring-boot:run     # Mac/Linux
apache-maven-3.9.6\bin\mvn.cmd spring-boot:run   # Windows
```

#### Step 4: Verify it started
Open your browser and go to: `http://localhost:8080/api/verify`  
You should see a JSON response (not a browser error page). If you see JSON, the backend is running correctly.

### Frontend Setup

#### Step 1: Navigate to frontend folder
```bash
cd frontend
```

#### Step 2: Create the environment file
Create a new file called `.env` in the `frontend/` folder and add this line:
```env
VITE_API_URL=http://localhost:8080/api
```
Save the file. (This tells the frontend where the backend API is.)

#### Step 3: Install dependencies
```bash
npm install
```
*(This will take 1-2 minutes on first run.)*

#### Step 4: Start the frontend
```bash
npm run dev
```

#### Step 5: Open the app
Go to `http://localhost:5173` in your browser. You should see the EduAuth Registry landing page.

### Test Accounts

| Role        | Email                    | Password     |
|-------------|--------------------------|--------------|
| Admin       | eduauthregistry@gmail.com      | admin123     |
| Student     | ssadidahmed01@gmail.com        | password123  |
| University  | admin@uiu.ac.bd           | password123  |
| Verifier    | demo@enosis.com     | password123  |

> All passwords are pre-hashed in the seed file. 
> Do not change them in the database directly — 
> use the forgot password flow if needed.

## Troubleshooting

### Issue: Backend fails to start with "Access denied for user 'root'"
**Fix:** Wrong MySQL password in `application.properties`. Double-check `spring.datasource.password` matches your MySQL password.

### Issue: Backend starts but emails are not being sent
**Fix:** Gmail App Password is wrong or 2-Step Verification is not enabled. Re-do the App Password steps in the backend setup above.

### Issue: Frontend shows "Network Error" or API calls fail
- **Fix 1:** Make sure the backend is running first (`http://localhost:8080`).
- **Fix 2:** Check that `frontend/.env` file exists and contains the correct URL.
- **Fix 3:** Make sure there are no spaces around the `=` sign in the `.env` file.

### Issue: "java: error: release version 17 not supported"
**Fix:** Your Java version is too old. Download Java 17 from: https://www.oracle.com/java/technologies/downloads/#java17

### Issue: MySQL tables not found on startup
**Fix:** Run the `schema.sql` file again. Make sure you ran it against the correct database (`eduauth_registry` not the default one).

### Issue: npm install fails with permission errors (Mac/Linux)
**Fix:** Do not use `sudo`. Instead run: `npm install --no-optional`

## Project Structure

```
eduauth-registry-maven/
├── backend/
│   ├── pom.xml
│   └── src/main/java/com/eduauth/
│       ├── controller/       # REST controllers (auth, student, university, verifier, admin, public)
│       ├── model/            # JPA entities
│       ├── repository/       # Spring Data JPA repositories
│       ├── service/          # Business logic
│       ├── dto/              # Request / response DTOs
│       └── config/           # Security, CORS, JWT configuration
├── frontend/
│   ├── src/
│   │   ├── pages/            # Route-level page components
│   │   ├── components/       # Shared UI components
│   │   └── api/              # Axios API client
│   └── package.json
├── db/
│   ├── schema.sql
│   └── seed.sql
├── API.md
└── LICENSE
```

## API Reference

See [API.md](API.md) for the complete API reference.

## License

Licensed under the MIT License. See [LICENSE](LICENSE).

## Team

**Team PhaseShift** — United International University
Advanced Object-Oriented Programming

| Member                  |
|-------------------------|
| Sadid Ahmed             |
| M.M. Sayem Prodhan      |
| Saikat Raihan           |
| Md. Mostafizur Rahman   |
