# Student Management System

A full-stack web application for managing student records with a clean architecture, built with Node.js/Express (Backend), React/Vite (Frontend), Microsoft SQL Server, and containerized with Docker Compose.

## Features

- 📋 Complete CRUD operations for student records
- 🏗️ Clean layered architecture (Controllers, Models, Routes, Config)
- 🐳 Docker Compose with Nginx reverse proxy and network isolation
- 🔐 Environment-based configuration (no hardcoded secrets)
- ⚡ Modern React UI with Vite build tooling
- 🧪 Production-ready project structure

---

## Quick Start

### Option 1: Docker Compose (Recommended)

**Prerequisites:** Docker and Docker Compose

1. **Create environment files:**

   `database.env`:

   ```env
   ACCEPT_EULA=Y
   MSSQL_PID=Developer
   MSSQL_SA_PASSWORD=YourStrong!Passw0rd
   ```

   `backend/.env`:

   ```env
   DB_HOST=database
   DB_PORT=1433
   DB_USER=sa
   DB_PASSWORD=YourStrong!Passw0rd
   DB_NAME=student_db
   DB_ENCRYPT=false
   DB_TRUST_SERVER_CERTIFICATE=true
   PORT=5000
   ```

2. **Launch containers:**

   ```bash
   docker compose up -d --build
   ```

3. **Access the app:**
   - Web UI: http://localhost:8080
   - Backend health check: http://localhost:8080/health

---

### Option 2: Local Development

**Prerequisites:** Node.js (LTS), Microsoft SQL Server

#### Database Setup

1. Execute `database_setup.sql` on your SQL Server instance.

#### Backend Setup

See [backend/README.md](backend/README.md) for detailed instructions.

```bash
cd backend
npm install
npm start
```

Backend runs on `http://localhost:5000`

#### Frontend Setup

See [frontend/README.md](frontend/README.md) for detailed instructions.

```bash
cd frontend
npm install
npm run dev
```

Frontend runs on `http://localhost:5173`

---

## Architecture Overview

### System Diagram

```
Host Machine (Port 8080)
      │
      ▼
[nginx (Gateway)]  ◄─── Bridge Network
      │
      ├─── (frontend_network) ───▶ [frontend - React + Nginx]
      │
      └─── (backend_network) ────▶ [backend - Express API]
                                     └───▶ [database - SQL Server]
```

### Network Isolation

- **Frontend Network:** Contains Nginx and frontend service
- **Backend Network:** Contains Nginx, backend service, and database
- **Nginx Role:** Only dual-homed service; acts as reverse proxy and API gateway
- **Database:** Only accessible from backend service

### Service Architecture

- **Frontend:** React SPA built with Vite, served by Nginx
- **Backend:** Express.js REST API with clean layered architecture
- **Database:** Microsoft SQL Server 2022
- **Gateway:** Nginx reverse proxy (routes `/api/*` → backend, everything else → frontend)

---

## Project Structure

```
.
├── README.md                      # Main documentation (this file)
├── README_DOCKER.md              # Docker Compose detailed docs (legacy, see README.md)
├── docker-compose.yml            # Docker Compose orchestration
├── database_setup.sql            # Database and table initialization
├── database.env                  # SQL Server container environment
│
├── backend/                      # Express.js API
│   ├── README.md                # Backend-specific documentation
│   ├── Dockerfile               # Backend container image
│   ├── package.json             # Dependencies
│   ├── .env                     # Backend environment variables
│   ├── server.js                # Express app entry point
│   └── src/
│       ├── config/db.js         # Database connection pool
│       ├── controllers/          # Request handlers
│       ├── models/               # Database queries
│       └── routes/               # API endpoint definitions
│
├── frontend/                     # React + Vite
│   ├── README.md                # Frontend-specific documentation
│   ├── Dockerfile               # Frontend build + Nginx serving
│   ├── package.json             # Dependencies
│   ├── vite.config.js           # Vite configuration
│   ├── eslint.config.js         # Linting rules
│   ├── nginx.conf               # Frontend Nginx config
│   ├── index.html               # HTML entry point
│   └── src/
│       ├── App.jsx              # Root component
│       ├── main.jsx             # React DOM mount
│       ├── index.css            # Global styles
│       ├── pages/               # Page components
│       │   ├── GetStudents.jsx  # List view
│       │   ├── CreateStudent.jsx# Create form
│       │   └── UpdateStudent.jsx# Update form
│       └── services/api.js      # Axios HTTP client
│
└── nginx/                        # Reverse proxy gateway
    ├── Dockerfile              # Nginx container image
    └── nginx.conf              # Proxy routing rules
```

---

## API Endpoints

All endpoints are relative to `/api` when running through Docker Compose gateway or direct backend.

### Students

| Method | Endpoint        | Description            |
| ------ | --------------- | ---------------------- |
| GET    | `/students`     | List all students      |
| POST   | `/students`     | Create a new student   |
| PUT    | `/students/:id` | Update a student by ID |
| DELETE | `/students/:id` | Delete a student by ID |

### Server Health

| Method | Endpoint  | Description                          |
| ------ | --------- | ------------------------------------ |
| GET    | `/health` | Server health check (returns 200 OK) |

### Request/Response Examples

**Create Student**

```bash
curl -X POST http://localhost:8080/api/students \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "department": "Computer Science"}'
```

Response (201):

```json
{
  "id": 1,
  "name": "John Doe",
  "department": "Computer Science",
  "message": "Student created successfully."
}
```

**Get All Students**

```bash
curl http://localhost:8080/api/students
```

Response (200):

```json
[
  {
    "id": 1,
    "name": "John Doe",
    "department": "Computer Science",
    "created_at": "2024-04-11T12:34:56.000Z"
  }
]
```

---

## Database Schema

**Database Name:** `student_db`

**Table: `students`**

| Column     | Type          | Constraints                |
| ---------- | ------------- | -------------------------- |
| id         | INT           | PRIMARY KEY, IDENTITY(1,1) |
| name       | NVARCHAR(255) | NOT NULL                   |
| department | NVARCHAR(MAX) | NOT NULL                   |
| created_at | DATETIME2     | DEFAULT SYSDATETIME()      |

**Initial Seed Data:**

- 10 sample students across various engineering departments
- Auto-inserted if table is empty during initialization

---

## Environment Variables

### Backend (`backend/.env`)

| Variable                      | Default               | Description             |
| ----------------------------- | --------------------- | ----------------------- |
| `DB_HOST`                     | `database`            | Database hostname       |
| `DB_PORT`                     | `1433`                | Database port           |
| `DB_USER`                     | `sa`                  | Database user           |
| `DB_PASSWORD`                 | `YourStrong!Passw0rd` | Database password       |
| `DB_NAME`                     | `student_db`          | Database name           |
| `DB_ENCRYPT`                  | `false`               | SQL Server encryption   |
| `DB_TRUST_SERVER_CERTIFICATE` | `true`                | Trust self-signed certs |
| `PORT`                        | `5000`                | Backend server port     |

### SQL Server Container (`database.env`)

| Variable            | Default               | Description            |
| ------------------- | --------------------- | ---------------------- |
| `ACCEPT_EULA`       | `Y`                   | Accept SQL Server EULA |
| `MSSQL_PID`         | `Developer`           | SQL Server edition     |
| `MSSQL_SA_PASSWORD` | `YourStrong!Passw0rd` | SA password            |

### Frontend (`frontend/`)

| Variable       | Default | Description                                                 |
| -------------- | ------- | ----------------------------------------------------------- |
| `VITE_API_URL` | `/api`  | API base URL (Compose), `http://localhost:5000/api` (local) |

---

## Development

### Backend Development

See [backend/README.md](backend/README.md)

- **Start server:** `npm start`
- **Lint:** Backend uses no lint script (consider adding ESLint)
- **Test:** No test suite yet (consider adding Jest)

### Frontend Development

See [frontend/README.md](frontend/README.md)

- **Dev server:** `npm run dev`
- **Build:** `npm run build`
- **Lint:** `npm run lint`
- **Preview:** `npm run preview`

### Docker Compose Commands

```bash
# Start services (detached, rebuild images)
docker compose up -d --build

# View running services
docker compose ps

# View logs
docker compose logs -f

# View logs for specific service
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f database

# Stop and remove containers (keep data volumes)
docker compose down

# Stop and remove containers and volumes
docker compose down -v

# Rebuild a specific service
docker compose up -d --build backend
```

---

## Credentials & Security

⚠️ **Important:**

- Environment files (`backend/.env`, `database.env`) are git-ignored and contain sensitive credentials.
- **Never commit these files!**
- Rotate default passwords in production.
- For production, use a secrets management solution (AWS Secrets Manager, Azure Key Vault, etc.).
- No authentication/authorization is currently implemented; add if needed.

---

## Ports Reference

| Service        | Container Port | Host Port | Access                 |
| -------------- | -------------- | --------- | ---------------------- |
| Nginx Gateway  | 80             | 8080      | http://localhost:8080  |
| Frontend Nginx | 80             | N/A       | Internal (via gateway) |
| Backend        | 5000           | N/A       | Internal (via gateway) |
| SQL Server     | 1433           | 1433      | http://localhost:1433  |

---

## Troubleshooting

### SQL Server fails to start

- **Symptom:** `database_init` service exits immediately
- **Fix:** Ensure `MSSQL_SA_PASSWORD` is strong (>= 8 chars, mixed case, numbers, symbols)

### Backend can't connect to database

- **Symptom:** Backend logs show connection errors
- **Check:**
  - `DB_HOST` is `database` (not `localhost`) in Docker
  - `MSSQL_SA_PASSWORD` matches in both `database.env` and `backend/.env`
  - `database_init` service completed successfully before backend started

### Frontend can't reach API

- **Symptom:** Requests to `/api/*` fail with 502 Bad Gateway
- **Check:**
  - Backend container is running (`docker compose ps`)
  - Nginx is correctly routing `/api/` to backend
  - Check Nginx logs: `docker compose logs nginx`

### Port conflicts

- **Symptom:** `docker compose up` fails with "port already allocated"
- **Fix:** Change the host port in `docker-compose.yml` or kill the process using the port:
  ```bash
  lsof -i :8080  # List process on port 8080
  kill -9 <PID>  # Kill process
  ```

---

## Next Steps / Roadmap

- [ ] Add authentication (JWT, session management)
- [ ] Add unit tests (Jest for backend and frontend)
- [ ] Add integration tests
- [ ] Add API versioning (`/api/v1/students`)
- [ ] Add input validation (Joi, Zod)
- [ ] Add request logging and monitoring
- [ ] Add health check endpoints to docker-compose
- [ ] Add CI/CD pipeline (GitHub Actions, Azure DevOps)
- [ ] Add container image scanning
- [ ] Implement database migrations framework
- [ ] Add API documentation (Swagger/OpenAPI)
- [ ] Add error tracking (Sentry, DataDog)

---

## License

ISC

---

## Additional Documentation

- [Backend README](backend/README.md) — Backend architecture, API details, setup
- [Frontend README](frontend/README.md) — Frontend architecture, component structure, setup
- `database_setup.sql` — Database initialization script
- `docker-compose.yml` — Container orchestration configuration
