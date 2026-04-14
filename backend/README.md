# Backend - Student Management System API

Express.js REST API for the Student Management System. Provides CRUD operations for student records with a clean layered architecture (Controllers, Models, Routes, Config).

## Overview

The backend is a lightweight Express.js server that:

- Exposes RESTful endpoints for student management
- Connects to Microsoft SQL Server via the `mssql` npm package
- Uses a connection pooling pattern for database access
- Implements proper HTTP status codes and error handling
- Follows separation of concerns (controllers, models, routes)

## Prerequisites

- **Node.js** (LTS recommended, tested with v18+)
- **Microsoft SQL Server** (local mode) or SQL Server container (Docker mode)

## Quick Start

### Local Development

1. **Environment Setup**

   Create/update `backend/.env`:

   ```env
   DB_HOST=localhost
   DB_PORT=1433
   DB_USER=sa
   DB_PASSWORD=YourStrong!Passw0rd
   DB_NAME=student_db
   DB_ENCRYPT=false
   DB_TRUST_SERVER_CERTIFICATE=true
   PORT=5000
   ```

   Or for Docker Compose mode, the env file differs (see below).

2. **Database Initialization**

   If you don't have the `student_db` database yet, run `database_setup.sql` on your SQL Server instance.

3. **Install Dependencies**

   ```bash
   npm install
   ```

4. **Start Server**

   ```bash
   npm start
   ```

   You should see:

   ```
   [dotenv@...] injecting env (8) from .env
   Server is running on port 5000
   ```

5. **Verify Health**

   ```bash
   curl http://localhost:5000/health
   ```

   Expected response:

   ```json
   {
     "status": "UP",
     "message": "Server is running"
   }
   ```

### Docker Compose Mode

When running via `docker compose up -d --build`, the backend container automatically:

- Receives `backend/.env` from the docker-compose.yml environment specification
- Connects to the `database` service (DNS resolution within Docker network)
- Waits for `database_init` to complete before starting
- Listens on port 5000 inside the container (exposed through Nginx gateway)

No extra setup needed—Docker Compose handles initialization.

## Project Structure

```
backend/
├── README.md                  # This file
├── Dockerfile                 # Container image
├── package.json              # npm dependencies & scripts
├── package-lock.json         # Locked versions
├── .env                      # Environment variables (git-ignored)
├── .gitignore               # Git ignore rules
├── server.js                # Express app entry point
└── src/
    ├── config/
    │   └── db.js            # Database connection pool config
    ├── controllers/
    │   └── studentController.js  # Request handlers
    ├── models/
    │   └── studentModel.js       # Database queries
    └── routes/
        └── studentRoutes.js      # Route definitions
```

## Architecture

### Layered Structure

```
Routes (HTTP endpoints)
   ↓
Controllers (Request handling & validation)
   ↓
Models (Database queries)
   ↓
Database Config (Connection pool & SQL execution)
   ↓
Microsoft SQL Server
```

### Key Files

**`server.js`**

- Express app initialization
- Middleware setup (CORS, JSON parsing)
- Route mounting
- Server startup

**`src/config/db.js`**

- Connection pool configuration
- Lazy pool initialization singleton
- Returns `getPool()` function and `sql` module
- Supports connection pooling with configurable limits

**`src/models/studentModel.js`**

- Static class with query methods
- Parameterized SQL queries (prevents SQL injection)
- Methods: `getAll()`, `create()`, `update()`, `delete()`
- Database-specific logic only; no business logic

**`src/controllers/studentController.js`**

- Request handlers for each endpoint
- Input validation
- Error handling with appropriate HTTP status codes
- Calls model layer for data operations

**`src/routes/studentRoutes.js`**

- Route definitions
- Maps HTTP methods and paths to controller methods
- Express router export

## API Endpoints

All endpoints serve requests at `http://localhost:5000` (local) or `http://localhost:8080/api` (Docker Compose).

### GET `/api/students`

List all students.

**Response (200 OK):**

```json
[
  {
    "id": 1,
    "name": "Alice Smith",
    "department": "Computer Science",
    "created_at": "2024-04-11T10:30:45.123Z"
  },
  {
    "id": 2,
    "name": "Bob Johnson",
    "department": "Mechanical Engineering",
    "created_at": "2024-04-11T10:30:46.456Z"
  }
]
```

### POST `/api/students`

Create a new student.

**Request Body:**

```json
{
  "name": "John Doe",
  "department": "Computer Science"
}
```

**Response (201 Created):**

```json
{
  "id": 3,
  "name": "John Doe",
  "department": "Computer Science",
  "message": "Student created successfully."
}
```

**Error (400 Bad Request):**
If `name` or `department` is missing:

```json
{
  "message": "Name and department are required."
}
```

### PUT `/api/students/:id`

Update a student by ID.

**Request Body:**

```json
{
  "name": "John Doe Updated",
  "department": "Software Engineering"
}
```

**Response (200 OK):**

```json
{
  "id": 3,
  "name": "John Doe Updated",
  "department": "Software Engineering",
  "message": "Student updated successfully."
}
```

**Error (404 Not Found):**
If student does not exist:

```json
{
  "message": "Student not found."
}
```

**Error (400 Bad Request):**
If `name` or `department` is missing:

```json
{
  "message": "Name and department are required."
}
```

### DELETE `/api/students/:id`

Delete a student by ID.

**Response (200 OK):**

```json
{
  "message": "Student deleted successfully."
}
```

**Error (404 Not Found):**
If student does not exist:

```json
{
  "message": "Student not found."
}
```

### GET `/health`

Health check endpoint.

**Response (200 OK):**

```json
{
  "status": "UP",
  "message": "Server is running"
}
```

## Environment Variables

| Variable                      | Required | Default      | Description                               |
| ----------------------------- | -------- | ------------ | ----------------------------------------- |
| `DB_HOST`                     | Yes      | —            | Database server hostname or IP            |
| `DB_PORT`                     | Yes      | `1433`       | Database server port                      |
| `DB_USER`                     | Yes      | —            | Database user (e.g., `sa`)                |
| `DB_PASSWORD`                 | Yes      | —            | Database password                         |
| `DB_NAME`                     | Yes      | `student_db` | Database name                             |
| `DB_ENCRYPT`                  | No       | `false`      | Enable SQL Server encryption              |
| `DB_TRUST_SERVER_CERTIFICATE` | No       | `true`       | Trust self-signed SQL Server certificates |
| `PORT`                        | No       | `5000`       | Express server port                       |

### Local vs. Docker Mode

**Local Development** (`backend/.env`):

```env
DB_HOST=localhost
DB_PORT=1433
DB_USER=sa
DB_PASSWORD=YourStrong!Passw0rd
DB_NAME=student_db
DB_ENCRYPT=false
DB_TRUST_SERVER_CERTIFICATE=true
PORT=5000
```

**Docker Compose** (set via `docker-compose.yml`):

```env
DB_HOST=database       # Use container name (DNS)
DB_PORT=1433
DB_USER=sa
DB_PASSWORD=YourStrong!Passw0rd
DB_NAME=student_db
DB_ENCRYPT=false
DB_TRUST_SERVER_CERTIFICATE=true
PORT=5000
```

## Dependencies

### Production

| Package   | Version | Purpose                      |
| --------- | ------- | ---------------------------- |
| `express` | ^5.2.1  | Web framework                |
| `cors`    | ^2.8.6  | CORS middleware              |
| `dotenv`  | ^17.3.1 | Environment variable loading |
| `mssql`   | ^11.0.1 | SQL Server driver            |

### Development

None currently configured. Consider adding:

- `eslint` for linting
- `jest` or `mocha` for testing
- `nodemon` for auto-restart during dev

## Scripts

```bash
# Start server (runs server.js)
npm start

# Add ESLint
npm install --save-dev eslint

# Add Jest for testing
npm install --save-dev jest

# Add Nodemon for auto-restart
npm install --save-dev nodemon
```

## Database Connection

The database connection uses a **lazy-loading singleton pattern**:

1. First request triggers `getPool()` call
2. `mssql.ConnectionPool` is created and connected
3. Pool is cached for subsequent requests
4. Pool reuses connections up to max (default 10)

### Connection Pool Config

```javascript
{
  max: 10,            // Max connections
  min: 0,             // Min connections
  idleTimeoutMillis: 30000  // 30 seconds idle timeout
}
```

To adjust, modify `src/config/db.js`.

## Error Handling

All endpoints return appropriate HTTP status codes:

| Status | Meaning      | Example                                           |
| ------ | ------------ | ------------------------------------------------- |
| 200    | OK           | GET successful, DELETE successful, PUT successful |
| 201    | Created      | POST successful                                   |
| 400    | Bad Request  | Missing required fields                           |
| 404    | Not Found    | ID doesn't exist                                  |
| 500    | Server Error | Unexpected database/server error                  |

All errors include a `message` field in the JSON response.

## Testing

Currently, no automated tests are configured. To add:

### Unit Tests with Jest

```bash
npm install --save-dev jest supertest
```

`package.json`:

```json
{
  "scripts": {
    "test": "jest --coverage"
  }
}
```

Example test:

```javascript
const request = require("supertest");
const app = require("../server");

describe("GET /health", () => {
  it("should return server status", async () => {
    const res = await request(app).get("/health");
    expect(res.status).toBe(200);
    expect(res.body.status).toBe("UP");
  });
});
```

## Linting

Currently, no linter is configured. To add:

```bash
npm install --save-dev eslint
npx eslint --init

# Then add to package.json:
"lint": "eslint ."
```

## Debugging

### Enable verbose logging

Modify `server.js` to log all requests:

```javascript
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});
```

### Check database connectivity

```bash
# From backend container, test SQL Server connection
sqlcmd -S database -U sa -P YourStrong!Passw0rd -Q "SELECT 1"
```

### View Docker logs

```bash
docker compose logs -f backend
```

## Performance Considerations

1. **Connection Pooling:** Already implemented with lazy loading. Adjust pool size if needed.
2. **Query Optimization:** `getAll()` sorts by ID. Consider adding pagination for large datasets.
3. **Caching:** No caching layer currently. Consider adding Redis for frequently accessed data.
4. **Input Validation:** Basic validation exists. Consider adding `joi` or `zod` for stricter validation.

## Security

⚠️ **Current Implementation:**

- No authentication or authorization
- Credentials in environment variables (safe in Docker, but not in git)
- CORS enabled for all origins (consider restricting in production)

### Recommended Additions

- [ ] Add JWT authentication
- [ ] Add input validation (Joi, Zod)
- [ ] Restrict CORS to specific origins
- [ ] Add rate limiting (express-rate-limit)
- [ ] Add SQL injection protection (already handled by parameterized queries)
- [ ] Add request logging (morgan)
- [ ] Add error tracking (Sentry)

## Next Steps

- [ ] Add unit tests
- [ ] Add ESLint configuration
- [ ] Add API documentation (Swagger/OpenAPI)
- [ ] Add input validation
- [ ] Add authentication
- [ ] Add request logging
- [ ] Add health checks for database connection
- [ ] Implement pagination for `GET /api/students`
- [ ] Add database migration framework
- [ ] Add rate limiting

## License

ISC

----------------------------------
Pipeline test 3