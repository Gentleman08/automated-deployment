# Frontend - Student Management System UI

React + Vite single-page application for the Student Management System. Provides a modern, component-driven UI for managing student records with client-side routing and HTTP API integration.

## Overview

The frontend is a React SPA that:

- Uses Vite for fast development and optimized production builds
- Implements client-side routing with React Router v6
- Communicates with the backend via Axios HTTP client
- Follows component-driven architecture with a services layer
- Includes ESLint for code quality

## Prerequisites

- **Node.js** (LTS recommended, tested with v18+)
- **npm** or similar package manager

## Quick Start

### Local Development

1. **Install Dependencies**

   ```bash
   npm install
   ```

2. **Start Development Server**

   ```bash
   npm run dev
   ```

   Output:

   ```
   VITE v5.4.21 ready in 123 ms

   ➜  Local:   http://localhost:5173/
   ```

3. **Open in Browser**

   Visit `http://localhost:5173`

   The development server watches files for changes and hot-reloads automatically.

### Production Build

1. **Build for Production**

   ```bash
   npm run build
   ```

   Output:

   ```
   ✓ 88 modules transformed
   dist/index.html                   0.47 kB
   dist/assets/index-DsIyL5lW.css    3.06 kB
   dist/assets/index-CfvbHkOq.js   205.96 kB
   ✓ built in 747ms
   ```

2. **Preview Build Locally**

   ```bash
   npm run preview
   ```

   Serves the production build on `http://localhost:4173`

### Docker Compose Mode

When running `docker compose up -d --build`:

- Frontend is built with `npm run build` during container creation
- Production build is served by Nginx on port 80 (inside container)
- Nginx serves static files and proxies `/api/*` requests to backend
- Accessible at `http://localhost:8080`

## Project Structure

```
frontend/
├── README.md                  # This file
├── Dockerfile                 # Docker build + serve
├── nginx.conf                 # Nginx config for serving
├── package.json              # npm dependencies & scripts
├── package-lock.json         # Locked versions
├── vite.config.js            # Vite configuration
├── eslint.config.js          # ESLint rules
├── index.html                # HTML entry point
├── .env                      # Environment variables (if needed)
├── .gitignore               # Git ignore rules
├── dist/                     # Production build output (generated)
├── public/                   # Static assets
└── src/
    ├── main.jsx              # React DOM entry point
    ├── App.jsx               # Root component with routing
    ├── index.css             # Global styles
    ├── assets/               # Images, icons, etc.
    ├── pages/                # Page components
    │   ├── GetStudents.jsx   # List students
    │   ├── CreateStudent.jsx # Create form
    │   └── UpdateStudent.jsx # Update form
    └── services/
        └── api.js            # Axios HTTP client
```

## Architecture

### Component Hierarchy

```
App (Router)
  ├── GetStudents (/)
  │   └── Student table with edit/delete actions
  ├── CreateStudent (/create)
  │   └── Form to add new student
  └── UpdateStudent (/update/:id)
      └── Form to edit existing student
```

### Data Flow

```
User Action (click, submit)
   ↓
Event Handler
   ↓
api.js (Axios call)
   ↓
Backend API (/api/students)
   ↓
State Update (setState)
   ↓
Re-render Component
   ↓
Display Updated UI
```

### Key Files

**`src/App.jsx`**

- Root component
- React Router setup with routes
- Navbar display
- Main layout

**`src/pages/GetStudents.jsx`**

- List all students with table
- Fetch data on mount
- Delete student with confirmation
- Link to create/update pages

**`src/pages/CreateStudent.jsx`**

- Form to add new student
- Form state management
- Submit handler
- Redirect to list on success

**`src/pages/UpdateStudent.jsx`**

- Form to edit student
- Pre-populate form with student data (via React Router state)
- Submit handler
- Redirect to list on success

**`src/services/api.js`**

- Axios HTTP client instance
- Base URL configuration
- Exported request functions
- Auto-handles JSON content-type

**`src/index.css`**

- Global CSS custom properties (variables)
- Component class styles
- Responsive design
- Color scheme

## Routes

| Path          | Component     | Purpose                    |
| ------------- | ------------- | -------------------------- |
| `/`           | GetStudents   | List all students          |
| `/create`     | CreateStudent | Form to create new student |
| `/update/:id` | UpdateStudent | Form to edit student by ID |

## Styling

Uses a custom CSS variable system for theming:

```css
:root {
  --primary: #4361ee;
  --primary-hover: #3a56d4;
  --secondary: #e2e8f0;
  --text-dark: #1e293b;
  --text-light: #64748b;
  --bg: #f8fafc;
  --surface: #ffffff;
  --border: #e2e8f0;
  --danger: #ef4444;
  --radius: 8px;
}
```

Change these variables to customize the entire app theme.

### Component Classes

- `.container` — Main card/container
- `.form-container` — Form wrapper
- `.btn` — Button base
- `.btn-primary` — Primary (blue) button
- `.btn-secondary` — Secondary (gray) button
- `.btn-danger` — Danger (red) button
- `.table` — Table styling
- `.form-group` — Form field wrapper
- `.form-control` — Input/textarea
- `.error-alert` — Error message display
- `.empty-state` — Empty list message
- `.navbar` — Top navigation bar

## Dependencies

### Production

| Package            | Version | Purpose             |
| ------------------ | ------- | ------------------- |
| `react`            | ^18.3.1 | UI library          |
| `react-dom`        | ^18.3.1 | React DOM rendering |
| `react-router-dom` | ^6.30.3 | Client-side routing |
| `axios`            | ^1.13.6 | HTTP client         |

### Development

| Package                | Version | Purpose                 |
| ---------------------- | ------- | ----------------------- |
| `vite`                 | ^5.4.21 | Build tool & dev server |
| `@vitejs/plugin-react` | ^4.3.3  | React JSX support       |
| `eslint`               | ^9.13.0 | Linter                  |
| `eslint-plugin-react`  | ^7.37.2 | React linting rules     |

## Environment Variables

### Development (`npm run dev`)

Default API base URL is `http://localhost:5000/api` (for local backend).

To override, create `.env` or `.env.local`:

```env
VITE_API_URL=http://custom-api.example.com/api
```

### Production / Docker Compose

Build-time environment variable (set during `npm run build`):

```bash
npm run build -- --mode production \
  --define "import.meta.env.VITE_API_URL='/api'"
```

Or via Docker `--build-arg`:

```dockerfile
ARG VITE_API_URL=/api
ENV VITE_API_URL=${VITE_API_URL}
RUN npm run build
```

Inside the container, Nginx proxies `/api/*` to the backend, so the frontend uses `/api` as the base URL.

## Scripts

```bash
# Start dev server on http://localhost:5173
npm run dev

# Build for production
npm run build

# Preview production build locally
npm run preview

# Lint code
npm run lint

# Lint and fix issues (if eslint-config supports it)
npm run lint -- --fix
```

## API Integration

The `src/services/api.js` file exports request functions:

```javascript
import {
  getStudents,
  createStudent,
  updateStudent,
  deleteStudent,
} from "./services/api";

// Get all students
const response = await getStudents();
console.log(response.data); // Array of students

// Create student
const response = await createStudent({ name: "John", department: "CS" });
console.log(response.data.id); // New student ID

// Update student
const response = await updateStudent(1, { name: "Jane", department: "EE" });

// Delete student
await deleteStudent(1);
```

### Error Handling

All requests can throw errors. Components wrap calls in try/catch:

```javascript
try {
  const response = await getStudents();
  setStudents(response.data);
} catch (error) {
  setError("Failed to fetch students. Please connect backend.");
}
```

Common errors:

- **Network Error**: Backend is not reachable
- **404 Not Found**: Student doesn't exist
- **400 Bad Request**: Invalid input data
- **500 Server Error**: Backend error

## Development Workflow

1. **Start dev server:**

   ```bash
   npm run dev
   ```

2. **Edit component files** (e.g., `src/pages/GetStudents.jsx`)

3. **Hot reload:**
   - Vite automatically refreshes browser on save
   - Component state may be lost (unless using React Fast Refresh)

4. **Check lint errors:**

   ```bash
   npm run lint
   ```

5. **Fix lint issues:**

   ```bash
   npm run lint -- --fix
   ```

6. **Build for production:**
   ```bash
   npm run build
   ```

## Form Handling

### CreateStudent & UpdateStudent

Both forms use React hooks for state:

```javascript
const [formData, setFormData] = useState({ name: "", department: "" });
const [error, setError] = useState("");
const [loading, setLoading] = useState(false);
```

Form submission:

1. Prevent default
2. Set loading state
3. Call API
4. Navigate on success
5. Set error message on failure

## Table Display (GetStudents)

The students table:

- Displays id, name, department, created_at
- Formats date using `new Date().toLocaleDateString()`
- Edit button links to `/update/:id` with student data in state
- Delete button confirms, then calls API

To pass data between routes:

```javascript
<Link to={`/update/${student.id}`} state={{ student }} className="btn">
  Edit
</Link>
```

Retrieve in destination:

```javascript
const location = useLocation();
const student = location.state?.student;
```

## Loading States

Components track loading state during async operations:

```javascript
const [loading, setLoading] = useState(false);

const handleSubmit = async (e) => {
  setLoading(true);
  try {
    await createStudent(formData);
  } finally {
    setLoading(false);
  }
};

<button disabled={loading}>{loading ? "Saving..." : "Save"}</button>;
```

## Error Handling

Error messages are displayed above forms:

```javascript
{
  error && <div className="error-alert">{error}</div>;
}
```

Backend error messages are extracted:

```javascript
catch (err) {
  setError(err.response?.data?.message || 'Failed to create student');
}
```

## Performance

### Optimizations

- **Lazy loading routes** (optional, via React.lazy):
  ```javascript
  const GetStudents = React.lazy(() => import("./pages/GetStudents"));
  ```
- **Vite code splitting** (automatic)
- **Production build minification** (automatic)
- **CSS bundling** (atomic, via Vite)

### No Optimizations Currently Needed

- Data set is small (students)
- No infinite scrolling needed
- No user-facing performance issues

If data grows:

- Implement pagination in backend
- Add page-based load in frontend
- Consider virtual scrolling for large tables

## Browser Support

Vite uses modern JavaScript (ES2020+). For older browser support, adjust `vite.config.js`:

```javascript
export default defineConfig({
  build: {
    target: "ES2015", // IE11 support (requires polyfills)
  },
});
```

## Testing

Currently, no automated tests are configured. To add:

### Unit/Component Tests with Vitest

```bash
npm install --save-dev vitest @testing-library/react
```

Example test:

```javascript
import { render, screen } from "@testing-library/react";
import GetStudents from "./pages/GetStudents";

test("renders students list", () => {
  render(<GetStudents />);
  expect(screen.getByText(/Students List/)).toBeInTheDocument();
});
```

## Docker Build

The `Dockerfile` uses a multi-stage build:

1. **Build stage:**
   - Uses `node:20-alpine`
   - Installs dependencies
   - Builds with `npm run build`

2. **Serve stage:**
   - Uses `nginx:alpine`
   - Copies built files to `/usr/share/nginx/html`
   - Copies `nginx.conf` for routing

This keeps the final image small (~50MB).

## Linting

Run ESLint to check code quality:

```bash
npm run lint
```

Current rules:

- React best practices
- React Hooks rules
- React Refresh support
- No unused variables

To fix auto-fixable issues:

```bash
npm run lint -- --fix
```

## Build Output

After `npm run build`, the `dist/` folder contains:

```
dist/
├── index.html           # Main entry point
├── assets/
│   ├── index-DsIyL5lW.css     # Minified CSS
│   └── index-CfvbHkOq.js      # Minified JS bundle
└── vite.svg
```

Total size: ~209 KB uncompressed, ~70 KB gzipped.

## Next Steps

- [ ] Add unit tests (Vitest + RTL)
- [ ] Add integration tests
- [ ] Add E2E tests (Playwright, Cypress)
- [ ] Add loading skeletons
- [ ] Add pagination
- [ ] Add search/filter
- [ ] Add sort functionality
- [ ] Add bulk operations
- [ ] Add date picker for filtering
- [ ] Add dark mode support
- [ ] Add accessibility improvements (a11y)
- [ ] Add form validation (Zod, Yup)
- [ ] Add toast notifications

## License

ISC

----------------------------------
Pipeline test 1