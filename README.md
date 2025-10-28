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

## 🏗️ Architecture

```

multi-tenant-notes-api/

multi-tenant-notes-api/
├── app/                              # Core application package
│   ├── main.py                       # FastAPI app entry point
│   ├── core/                         # Core utilities
│   │   ├── config.py                 # Application settings (Pydantic Settings)
│   │   ├── security.py               # Password hashing (bcrypt + SHA-256) & JWT token generation
│   │   └── auth.py                   # JWT decoding, user validation, and tenant isolation logic
│   ├── api/                          # API layer
│   │   ├── deps.py                   # Dependency injections (e.g., RBAC via `require_role`)
│   │   └── v1/                       # Versioned API
│   │       ├── router.py             # Main API router
│   │       └── routes/               # Route handlers
│   │           ├── organizations.py  # POST /organizations/
│   │           ├── users.py          # POST /organizations/{id}/users/ + /login
│   │           └── notes.py          # Notes CRUD with RBAC
│   ├── models/                       # Pydantic models with MongoDB ObjectId support
│   │   ├── organization.py
│   │   ├── user.py
│   │   └── note.py
│   ├── schemas/                      # Input validation schemas (DTOs)
│   │   ├── organization.py
│   │   ├── user.py
│   │   ├── auth.py                   # ← Login request schema (UserLogin)
│   │   └── note.py
│   ├── services/                     # Business logic layer
│   │   ├── organization_service.py
│   │   ├── user_service.py           # User creation & authentication
│   │   └── note_service.py
│   └── db/                           # Database layer
│       └── session.py                # Async Motor (MongoDB) client
├── tests/                            # Automated tests
│   └── test_notes.py                 # Full RBAC & multi-tenancy test suite
├── .env                              # Environment variables (optional)
├── requirements.txt                  # Python dependencies
├── Dockerfile                        # Container build definition
├── docker-compose.yml                # Local dev with MongoDB
├── README.md                         # Setup, usage, and examples
└── pytest.ini                        # Test configuration (async mode)
```

## 🚀 Quick Start

### Option 1: Docker

```bash
# Clone the repository
git clone <your-repo-url>
cd Multi-Tenant Notes API

# Start services (API + MongoDB)
docker-compose up --build

# Access the API
curl http://localhost:8000/
```

The API will be available at `http://localhost:8000`.

### Option 2: Local Development

**Prerequisites:**
- Python 3.11+
- MongoDB 8.0+ running locally

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

### Organizationshttp://localhost:8000/api/v1/organizations/

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/organizations` | Create organization | No |

### Userslocalhost:8000/api/v1/organizations/<ORG_ID>/users/

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/api/v1/organizations/{org_id}/users` | Create user | No |

### Notes

| Method | Endpoint | Description | Auth Required | Roles |
|--------|----------|-------------|---------------|-------|
| POST | `/api/v1/notes` | Create note | Yes | writer, admin |
| GET | `/api/v1/notes` | List notes | Yes | all |
| GET | `/api/v1/notes/{id}` | Get specific note | Yes | all |
| DELETE | `/api/v1/notes/{id}` | Delete note | Yes | admin only |

## 🔐 Role Permissions

| Role | View Notes | Create Notes | Update Notes | Delete Notes |
|------|------------|--------------|--------------|--------------|
| **reader** | ✅ | ❌ | ❌ | ❌ |
| **writer** | ✅ | ✅ | ✅ | ❌ |
| **admin** | ✅ | ✅ | ✅ | ✅ |

## 📝 Usage Examples

## 💡 Note About Examples

The IDs in these examples (like `65f1a2b3c4d5e6f7g8h9i0j1`) are placeholders. 
When you run the commands:

1. **Save the response IDs** from create operations
2. **Use those actual IDs** in subsequent commands
3. **Replace placeholders** with your real values

Example workflow:

### 1. Create an Organization

```bash
curl -X POST "http://localhost:8000/api/v1/organizations" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Samad Limited",
  }'

# Response:
# {
#   "id": "65f1a2b3c4d5e6f7g8h9i0j1",
#   "name": "Samad Limited",
# }
```

### 2. Create Users

```bash
# Create an admin user
ORG_ID="65f1a2b3c4d5e6f7g8h9i0j1"

curl -X POST "http://localhost:8000/api/v1/organizations/${ORG_ID}/users" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@samad.com",
    "password": "SecurePass123!",
    "role": "admin"
  }'

# Response:
# {
#  "user_id": "65f1a2b3c4d5e6f7g8h9i0j1",
#   "email": "admin@samad.com",
#   "role": "admin",
#   "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
# }

# Create a writer
curl -X POST "http://localhost:8000/api/v1/organizations/${ORG_ID}/users" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "writer@samad.com",
    "password": "SecurePass123!",
    "role": "writer"
  }'

# Response:
# {
#  "user_id": "65f1a2b3c4d5e6f7g8h9i0j1",
#   "email": "writer@samad.com",
#   "role": "writer",
#   "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
# }

# Create a reader
curl -X POST "http://localhost:8000/api/v1/organizations/${ORG_ID}/users" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "reader@samad.com",
    "password": "SecurePass123!",
    "role": "reader",
  }'

# Response:
# {
#  "user_id": "65f1a2b3c4d5e6f7g8h9i0j1",
#   "email": "reader@samad.com",
#   "role": "reader",
#   "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
# }

# Save token for subsequent requests
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```


### 3. Create a Note (as writer)

```bash
curl -X POST "http://localhost:8000/api/v1/notes" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Project Planning",
    "content": "Initial project requirements and timeline"
  }'

# Response:
# {
#   "id": "65f1a2b3c4d5e6f7g8h9i0j1",
#   "title": "Project Planning",
# }
```

### 4. List Notes

```bash
# List all notes in the organization
curl -X GET "http://localhost:8000/api/v1/notes" \
  -H "Authorization: Bearer ${TOKEN}"

# Response:
[
# {
#   "id": "65f1a2b3c4d5e6f7g8h9i0j1",
#   "title": "Project Planning",
# }
]
```

### 5. Get Specific Note

```bash
NOTE_ID="65f1a2b3c4d5e6f7g8h9i0j2"

curl -X GET "http://localhost:8000/api/v1/notes/${NOTE_ID}" \
  -H "Authorization: Bearer ${TOKEN}"

# Response:
# {
#   "id": "65f1a2b3c4d5e6f7g8h9i0j1",
#   "title": "Project Planning",
#   "content": "Initial project requirements and timeline"
# }
```

### 8. Delete a Note (admin only)

```bash
# Delete the note
curl -X DELETE "http://localhost:8000/api/v1/notes/${NOTE_ID}" \
  -H "Authorization: Bearer ${ADMIN_TOKEN}"

# Response:
# {
#   "detail": "Note deleted"
# }
```

## 🧪 Running Tests

```bash
# Run all tests
pytest tests/ -v

# Run specific test
pytest tests/test_api
