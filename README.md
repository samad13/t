# Multi-Tenant Notes API

A production-ready, multi-tenant Notes API built with FastAPI and MongoDB, featuring JWT authentication and role-based access control (RBAC).

## 🎯 Features

- **Multi-Tenancy**: Complete data isolation between organizations
- **Role-Based Access Control**: Three roles (reader, writer, admin) with distinct permissions
- **JWT Authentication**: Secure token-based authentication
- **Clean Architecture**: Separated layers (routers, models, dependencies, middleware)
- **Async Database Operations**: Using Motor for MongoDB async queries
- **Comprehensive Testing**: Automated tests for CRUD and permissions
- **Docker Support**: Complete containerization setup
- **API Documentation**: Auto-generated OpenAPI/Swagger docs

## 🏗️ Architecture

```
.
├── main.py                          # FastAPI application entry point
├── app/
│   ├── models.py                    # Pydantic models for requests/responses
│   ├── database.py                  # MongoDB connection layer
│   ├── routers/
│   │   ├── organizations.py         # Organization endpoints
│   │   ├── users.py                 # User management & auth
│   │   └── notes.py                 # Notes CRUD with RBAC
│   ├── dependencies/
│   │   └── auth.py                  # Authentication dependencies
│   ├── middleware/
│   │   └── tenant.py                # Tenant context middleware
│   └── utils/
│       └── auth.py                  # JWT & password utilities
├── tests/
│   └── test_api.py                  # Automated API tests
├── docker-compose.yml               # Docker orchestration
├── Dockerfile                       # Container definition
├── requirements.txt                 # Python dependencies
└── README.md                        # This file
```

## 🚀 Quick Start

### Option 1: Docker (Recommended)

```bash
# Clone the repository
git clone <your-repo-url>
cd notes-api

# Start services (API + MongoDB)
docker-compose up -d

# Check logs
docker-compose logs -f api

# Access the API
curl http://localhost:8000/health
```

The API will be available at `http://localhost:8000` with interactive docs at `http://localhost:8000/docs`.

### Option 2: Local Development

**Prerequisites:**
- Python 3.11+
- MongoDB 7.0+ running locally

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set environment variables
export MONGODB_URL="mongodb://localhost:27017"
export SECRET_KEY="your-secret-key-here"

# Run the application
uvicorn main:app --reload

# In a new terminal, run tests
pytest tests/ -v
```

## 📡 API Endpoints

### Organizations

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/organizations` | Create organization | No |
| GET | `/api/v1/organizations/{org_id}` | Get organization details | No |
| GET | `/api/v1/organizations` | List organizations | No |

### Users

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/organizations/{org_id}/users` | Create user | No |
| POST | `/api/v1/auth/login` | Login and get token | No |
| GET | `/api/v1/users/me` | Get current user | Yes |
| GET | `/api/v1/organizations/{org_id}/users` | List org users | Yes |

### Notes

| Method | Endpoint | Description | Auth Required | Roles |
|--------|----------|-------------|---------------|-------|
| POST | `/api/v1/notes` | Create note | Yes | writer, admin |
| GET | `/api/v1/notes` | List notes | Yes | all |
| GET | `/api/v1/notes/{id}` | Get specific note | Yes | all |
| PUT | `/api/v1/notes/{id}` | Update note | Yes | writer, admin |
| DELETE | `/api/v1/notes/{id}` | Delete note | Yes | admin only |

## 🔐 Role Permissions

| Role | View Notes | Create Notes | Update Notes | Delete Notes |
|------|------------|--------------|--------------|--------------|
| **reader** | ✅ | ❌ | ❌ | ❌ |
| **writer** | ✅ | ✅ | ✅ | ❌ |
| **admin** | ✅ | ✅ | ✅ | ✅ |

## 📝 Usage Examples

### 1. Create an Organization

```bash
curl -X POST "http://localhost:8000/api/v1/organizations" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme Corp",
    "description": "A demo organization"
  }'

# Response:
# {
#   "id": "65f1a2b3c4d5e6f7g8h9i0j1",
#   "name": "Acme Corp",
#   "description": "A demo organization",
#   "created_at": "2024-01-15T10:30:00Z"
# }
```

### 2. Create Users

```bash
# Create an admin user
ORG_ID="65f1a2b3c4d5e6f7g8h9i0j1"

curl -X POST "http://localhost:8000/api/v1/organizations/${ORG_ID}/users" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@acme.com",
    "password": "SecurePass123!",
    "role": "admin",
    "full_name": "Admin User"
  }'

# Create a writer
curl -X POST "http://localhost:8000/api/v1/organizations/${ORG_ID}/users" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "writer@acme.com",
    "password": "SecurePass123!",
    "role": "writer",
    "full_name": "Writer User"
  }'

# Create a reader
curl -X POST "http://localhost:8000/api/v1/organizations/${ORG_ID}/users" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "reader@acme.com",
    "password": "SecurePass123!",
    "role": "reader",
    "full_name": "Reader User"
  }'
```

### 3. Login and Get Token

```bash
curl -X POST "http://localhost:8000/api/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "writer@acme.com",
    "password": "SecurePass123!"
  }'

# Response:
# {
#   "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
#   "token_type": "bearer"
# }

# Save token for subsequent requests
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### 4. Create a Note (as writer)

```bash
curl -X POST "http://localhost:8000/api/v1/notes" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Project Planning",
    "content": "Initial project requirements and timeline",
    "tags": ["planning", "project"]
  }'
```

### 5. List Notes

```bash
# List all notes in the organization
curl -X GET "http://localhost:8000/api/v1/notes" \
  -H "Authorization: Bearer ${TOKEN}"

# Filter by tag
curl -X GET "http://localhost:8000/api/v1/notes?tag=planning" \
  -H "Authorization: Bearer ${TOKEN}"

# With pagination
curl -X GET "http://localhost:8000/api/v1/notes?skip=0&limit=10" \
  -H "Authorization: Bearer ${TOKEN}"
```

### 6. Get Specific Note

```bash
NOTE_ID="65f1a2b3c4d5e6f7g8h9i0j2"

curl -X GET "http://localhost:8000/api/v1/notes/${NOTE_ID}" \
  -H "Authorization: Bearer ${TOKEN}"
```

### 7. Update a Note

```bash
curl -X PUT "http://localhost:8000/api/v1/notes/${NOTE_ID}" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Project Planning",
    "content": "Updated requirements with feedback",
    "tags": ["planning", "project", "updated"]
  }'
```

### 8. Delete a Note (admin only)

```bash
# First, login as admin
ADMIN_TOKEN=$(curl -s -X POST "http://localhost:8000/api/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@acme.com", "password": "SecurePass123!"}' \
  | jq -r '.access_token')

# Delete the note
curl -X DELETE "http://localhost:8000/api/v1/notes/${NOTE_ID}" \
  -H "Authorization: Bearer ${ADMIN_TOKEN}"
```

## 🧪 Running Tests

```bash
# Run all tests
pytest tests/ -v

# Run specific test
pytest tests/test_api
