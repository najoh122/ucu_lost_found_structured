
# UCU Campus Lost & Found

A Responsive web application for reporting, searching and reclaiming lost items on the UCU campus.
Built for **CSC1202 – Web and Mobile Applications Development** (Advent 2026).

- **Students** report lost or found items, search listings, submit private ownership claims, and track their own reports and claims.
- **Administrators** review claims, approve or reject them with a reason, confirm when an item has been returned, manage listings, and create other administrator accounts.

Private identifying details and claim evidence are protected **on the backend**. They are never included in public responses.

- **Deployed application:** _add URL here_
- **Repository:** https://github.com/aaronwadeje/ucu_lost_found_structured
- **Final commit ID:** _add here before submission_

---

## Table of contents

1. [Tech stack](#tech-stack)
2. [Project structure](#project-structure)
3. [Prerequisites](#prerequisites)
4. [Setup](#setup)
5. [Administrator accounts](#administrator-accounts)
6. [Production build](#production-build)
7. [How the claim workflow works](#how-the-claim-workflow-works)
8. [API reference](#api-reference)
9. [Frontend folder map](#frontend-folder-map)
10. [Security notes](#security-notes)
11. [Troubleshooting](#troubleshooting)
12. [Acknowledgements](#acknowledgements)
13. [Authors](#authors)

---

## Tech stack

| Layer    | Technology                                                                     |
|----------|--------------------------------------------------------------------------------|
| Frontend | React 18, React Router 6, Vite 5                                               |
| Backend  | Node.js, Express 4                                                             |
| Database | MySQL                                                                          |
| Security | bcrypt password hashing (`bcryptjs`), JWT sessions, token revocation on logout |
| Uploads  | Multer (images only, 5 MB limit)                                               |

## Project structure

```
ucu_lost_found_structured/
├── backend/
│   ├── server.js          # Express app and all API routes
│   ├── db.js              # MySQL connection pool
│   ├── schema.sql         # Database setup script (creates database + tables)
│   ├── seed.js            # Optional: creates fictional sample student accounts
│   ├── create-admin.js    # Creates / updates the first administrator from .env
│   ├── .env.example       # Template for your private .env file
│   └── uploads/           # Uploaded photos (contents are not committed)
└── frontend/
    ├── src/               # React source (pages, components, api.js)
    ├── vite.config.js     # Dev server + proxy to the backend
    └── package.json
```

## Prerequisites

- **Node.js 18 or newer** (check with `node -v`)
- **MySQL 8** running locally, with a user that can create databases
- **Git**

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/aaronwadeje/ucu_lost_found_structured.git
cd ucu_lost_found_structured
```

### 2. Create the database

From the `backend` folder, run the setup script. This works in Command Prompt, PowerShell and Git Bash:

```bash
cd backend
mysql -u root -p -e "source schema.sql"
```

Enter your MySQL password when asked. This creates the `ucu_lost_found` database and its tables
(`users`, `items`, `claims`, `revoked_tokens`).

> Prefer a GUI? Open `schema.sql` in MySQL Workbench and run the whole script.

### 3. Configure the backend (`.env` file)

The `.env` file holds your private settings. **It must never be committed to Git.**
It is already listed in `.gitignore`.

1. In the `backend` folder, copy the template to a new file named `.env`:

   ```bash
   # Windows (Command Prompt)
   copy .env.example .env

   # Windows (PowerShell), macOS, Linux, Git Bash
   cp .env.example .env
   ```

2. Open `backend/.env` in VS Code (or any text editor) and fill in the values:

   ```env
   PORT=5000

   DB_HOST=localhost
   DB_USER=root
   DB_PASSWORD=your_mysql_password
   DB_NAME=ucu_lost_found

   JWT_SECRET=paste_a_long_random_string_here

   ADMIN_NAME=Your Full Name
   ADMIN_EMAIL=admin@example.com
   ADMIN_PASSWORD=a-unique-password-of-12-or-more-characters
   ```

| Variable         | What  is this                                                            |
|------------------|------------------------------------------------------------------------|
| `PORT`           | Port the backend listens on (default `5000`).                          |
| `DB_HOST`        | MySQL host (usually `localhost`).                                      |
| `DB_USER`        | MySQL username.                                                        |
| `DB_PASSWORD`    | MySQL password.                                                        |
| `DB_NAME`        | Database name (`ucu_lost_found`).                                      |
| `JWT_SECRET`     | Secret used to sign login tokens. **Set your own long random value.**  |
| `ADMIN_NAME`     | Full name of the first administrator.                                  |
| `ADMIN_EMAIL`    | Login email of the first administrator.                                |
| `ADMIN_PASSWORD` | **Initial administrator password.** Minimum **12 characters**.         |

Generate a strong `JWT_SECRET` with:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

> If `JWT_SECRET` is missing, the server falls back to an insecure development default.
> Always set your own value.

### 4. Install backend dependencies and create the administrator

```bash
npm install
npm run admin:create
```

You should see: `Administrator account provisioned for <your email>.`

This reads `ADMIN_NAME`, `ADMIN_EMAIL` and `ADMIN_PASSWORD` from `.env`, hashes the password with bcrypt, and stores the account with the `admin` role.

Notes:
- If you see an error such as *"ADMIN_PASSWORD must be at least 12 characters"*, fix the value in `.env` and run the command again.
- Running the command again for the same email **resets that administrator's password** to the current `ADMIN_PASSWORD`.
- After the account exists, you can delete the `ADMIN_PASSWORD` line from `.env`. Change your password later from the admin dashboard.

### 5. (Optional) Add fictional sample students

```bash
npm run seed
```

This creates two fictional student accounts for testing. The password for both is `password123`.

| Name          | Email                       | Role    |
|---------------|-----------------------------|---------|
| Namusoke Ruth | namusoke@students.ucu.ac.ug | student |
| Okello James  | okello@students.ucu.ac.ug   | student |

> Never run `npm run seed` against a production database, and never use `password123` on a deployed system.

### 6. Start the backend

```bash
npm start
```

The API runs at **http://localhost:5000**. Keep this terminal open.

### 7. Start the frontend

Open a **second terminal**:

```bash
cd frontend
npm install
npm run dev
```

Open **http://localhost:5173** in your browser. In development, Vite forwards `/api` and `/uploads` requests to the backend automatically (configured in `vite.config.js`).

### 8. Sign in as the administrator

Use the **Admin sign in** page with the `ADMIN_EMAIL` and `ADMIN_PASSWORD` you set in step 3.

---

## Administrator accounts

- Public registration **always** creates a **student** account.
- The **first** administrator is provisioned from the command line (`npm run admin:create`, see [Setup step 4](#4-install-backend-dependencies-and-create-the-administrator)).
- An existing administrator creates each additional admin from the protected **Administrators** section of the dashboard, giving each their own name, email and unique initial password (minimum 12 characters). Share the initial password securely.
- Each administrator can update their own name, email and password from the dashboard. Changing profile details requires the current password.

## Production build

```bash
cd frontend
npm run build
```

The build is written to `frontend/dist`. For a deployed build, set `VITE_API_URL` to your backend's public API URL (for example `https://your-backend.example.com/api`) before building. See `frontend/.env.example`.

---

## How the claim workflow works

1. A student reports an item as **lost** or **found**. For found items, private identifying details go in a separate field visible only to the reporter and administrator.
2. Anyone can search and filter listings by keyword, category, location, date and type. Each listing shows whether it is **open** or **returned**.
3. A logged-in student claims a found item by privately explaining why it is theirs. Students cannot claim items they reported, items already returned, or the same item twice.
4. The administrator **approves or rejects** the claim and records a reason. Only one claim can be approved per item.
5. After the owner collects the item, the administrator marks it **returned** (only possible after an approved claim). No further claims are accepted after that.

**Statuses**

- Item: `open`, `returned`
- Claim: `pending`, `approved`, `rejected`

---

## API reference

Protected routes expect `Authorization: Bearer <token>`.
Errors follow the shape `{ "message": "...", "errors": { "field": "..." } }`.

### Authentication

| Method & path             | Access    | Body                      | Returns                           |
|---------------------------|-----------|---------------------------|-----------------------------------|
| `POST /api/auth/register` | Public    | name, email, password     | `{ token, user:{id,name,role} }`  |
| `POST /api/auth/login`    | Public    | email, password           | `{ token, user }`                 |
| `POST /api/auth/logout`   | Logged in | –                         | `{ ok:true }` (token is revoked)  |

### Items and claims (students)

| Method & path                  | Access    | Body / query                                                                    | Returns                                                                                                                                                      |
|--------------------------------|-----------|---------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `GET /api/items`               | Public    | query: `q`, `type`, `category`, `location`, `date`, `status`                    | Array of public items: id, type, title, category, description, date, location, photo_url, status. **No private details.**                                    |
| `GET /api/items/:id`           | Public*   | –                                                                               | Public item + `is_owner`. Owner/admin also get `private_details`. A logged-in claimant also gets `my_claim:{id,status,evidence,admin_reason}`.               |
| `POST /api/items`              | Logged in | multipart: type, title, category, description, date, location, private_details?, photo? | `{ id }`                                                                                                                                              |
| `PUT /api/items/:id`           | Owner or admin | same as above                                                              | `{ id }`                                                                                                                                                     |
| `GET /api/my/items`            | Logged in | –                                                                               | Array of your own items                                                                                                                                      |
| `POST /api/items/:id/claims`   | Logged in | `{ evidence }`                                                                  | Created claim (rejects own item, returned item, duplicate claim)                                                                                             |
| `GET /api/my/claims`           | Logged in | –                                                                               | Array: id, item_id, item_title, status, admin_reason, created_at                                                                                             |

\* Private fields are only returned to the item's owner and administrators.

### Administration (admin only)

| Method & path                      | Body                                          | Returns                                                                                                                   |
|------------------------------------|-----------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| `GET /api/admin/items`             | –                                             | Array of listings with reporter names                                                                                     |
| `DELETE /api/admin/items/:id`      | –                                             | Permanently deletes a listing and its claims                                                                              |
| `GET /api/admin/claims`            | –                                             | Array: id, item_id, item_title, item_status, item_private_details, claimant_name, evidence, status, admin_reason, created_at |
| `PATCH /api/admin/claims/:id`      | `{ decision:'approved'｜'rejected', reason }` | Updated claim (only one approved claim per item)                                                                          |
| `POST /api/admin/items/:id/return` | –                                             | Item marked `returned` (only after an approved claim)                                                                     |
| `GET /api/admin/users`             | –                                             | Administrator account list                                                                                                |
| `POST /api/admin/users`            | name, email, password                         | Created administrator account                                                                                             |
| `GET /api/admin/profile`           | –                                             | Current administrator account                                                                                             |
| `PATCH /api/admin/profile`         | name, email, currentPassword, newPassword?    | Updated current administrator account                                                                                     |

---

## Frontend folder map

- `src/api.js` – every call to the backend (one place to change URLs)
- `src/AuthContext.jsx` – login state, token storage, logout
- `src/pages/` – Login, Home, ReportForm (create + edit), ItemDetail (claim form), MyActivity, Admin
- `src/components/` – Navbar, ItemCard, Field, StatusBadge, Feedback (loading/empty/alerts), ProtectedRoute
- `src/styles.css` – all styling (mobile first)

## Security notes

- Passwords are stored only as bcrypt hashes.
- Permissions are enforced on the backend; hiding buttons or pages in React is only a convenience.
- Private details and claim evidence are never sent to people who should not see them.
- Logging out revokes the token on the server, so it stops working immediately.
- Secrets live in `backend/.env`, which is excluded from Git. Never commit it.
- Never run `npm run seed` or use `password123` on a deployed system.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `Database initialization failed` | Check `DB_*` values in `backend/.env` and make sure MySQL is running and `schema.sql` was run. |
| `Access denied for user` | The `DB_USER` / `DB_PASSWORD` in `.env` do not match your MySQL login. |
| `Set ADMIN_NAME ...` / `ADMIN_PASSWORD must be at least 12 characters` | Fill in all three `ADMIN_*` values correctly in `backend/.env`, then rerun `npm run admin:create`. |
| Frontend loads but shows network errors | Confirm the backend is running on port 5000 (`npm start` in `backend`). |
| Port already in use | Change `PORT` in `backend/.env`, and update the proxy target in `frontend/vite.config.js` to match. |
| `mysql` is not recognised | Add the MySQL `bin` folder to your PATH, or run `schema.sql` in MySQL Workbench. |

## Acknowledgements

_List external code, references and any permitted AI assistance here, as required by the examination instructions._

## Authors

_Add team members' names and contributions here._
