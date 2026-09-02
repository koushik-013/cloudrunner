# CloudRunner Backend

A robust REST API backend built with Axum for managing university teachers and their NixOS configuration files.

## Features

- **Authentication System**
  - User registration with email validation
  - Login with JWT token generation
  - Logout (stateless JWT)
  - Password reset flow with token-based confirmation

- **NixOS Configuration Management**
  - Upload `.nix` configuration files
  - List all configuration files
  - Get specific configuration file
  - Edit configuration file content
  - Delete configuration files

- **Security**
  - Password hashing with bcrypt
  - JWT-based authentication
  - Token expiration
  - Password reset tokens with expiration

## Tech Stack

- **Framework**: Axum 0.7
- **Database**: PostgreSQL with SQLx
- **Authentication**: JWT (jsonwebtoken)
- **Password Hashing**: bcrypt
- **Runtime**: Tokio

## Prerequisites

- Rust 1.70+ (install from [rustup.rs](https://rustup.rs/))
- PostgreSQL 14+
- sqlx-cli (for database migrations)

## Setup Instructions

### 1. Install Dependencies

```bash
# Install Rust (if not already installed)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install sqlx-cli for database migrations
cargo install sqlx-cli --no-default-features --features postgres
```

### 2. Database Setup

```bash
# Create PostgreSQL database
createdb university_nixos

# Or using psql
psql -U postgres -c "CREATE DATABASE university_nixos;"

# Run the schema
psql -U postgres -d university_nixos -f schema.sql
```

### 3. Environment Configuration

Create a `.env` file in the project root:

```bash
cp .env.example .env
```

Edit `.env` with your configuration:

```env
DATABASE_URL=postgres://username:password@localhost:5432/university_nixos
JWT_SECRET=your-secret-key-change-this-in-production-use-strong-random-string
JWT_EXPIRATION_HOURS=24
SERVER_HOST=127.0.0.1
SERVER_PORT=8080
```

**Security Note**: Generate a strong JWT secret:
```bash
openssl rand -base64 32
```

### 4. Build and Run

```bash
# Build the project
cargo build --release

# Run the server
cargo run --release
```

The server will start on `http://127.0.0.1:8080`

## API Endpoints

### Authentication

#### Register
```http
POST /api/auth/register
Content-Type: application/json

{
  "email": "teacher@university.edu",
  "password": "securepassword123",
  "full_name": "John Doe",
  "department": "Computer Science"
}
```

#### Login
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "teacher@university.edu",
  "password": "securepassword123"
}
```

Response:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "teacher": {
      "id": "uuid",
      "email": "teacher@university.edu",
      "full_name": "John Doe",
      "department": "Computer Science"
    }
  }
}
```

#### Logout
```http
POST /api/auth/logout
Authorization: Bearer <token>
```

#### Request Password Reset
```http
POST /api/auth/reset-password
Content-Type: application/json

{
  "email": "teacher@university.edu"
}
```

#### Confirm Password Reset
```http
POST /api/auth/reset-password/confirm
Content-Type: application/json

{
  "token": "reset-token-from-email",
  "new_password": "newSecurePassword123"
}
```

#### Get Current Teacher
```http
GET /api/auth/me
Authorization: Bearer <token>
```

### NixOS Configuration Management

All config endpoints require authentication via Bearer token.

#### Upload Configuration
```http
POST /api/configs
Authorization: Bearer <token>
Content-Type: application/json

{
  "filename": "configuration.nix",
  "content": "{ config, pkgs, ... }:\n\n{\n  # Your NixOS config here\n}"
}
```

#### List All Configurations
```http
GET /api/configs
Authorization: Bearer <token>
```

#### Get Specific Configuration
```http
GET /api/configs/{config_id}
Authorization: Bearer <token>
```

#### Update Configuration
```http
PUT /api/configs/{config_id}
Authorization: Bearer <token>
Content-Type: application/json

{
  "content": "{ config, pkgs, ... }:\n\n{\n  # Updated config\n}"
}
```

#### Delete Configuration
```http
DELETE /api/configs/{config_id}
Authorization: Bearer <token>
```

## Testing with curl

### Register a new teacher
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@university.edu",
    "password": "password123",
    "full_name": "Test Teacher",
    "department": "Computer Science"
  }'
```

### Login
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@university.edu",
    "password": "password123"
  }'
```

### Upload a config (replace TOKEN)
```bash
curl -X POST http://localhost:8080/api/configs \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "configuration.nix",
    "content": "{ config, pkgs, ... }:\n{\n  system.stateVersion = \"23.11\";\n}"
  }'
```

## Database Schema

### Teachers Table
- `id`: UUID (primary key)
- `email`: VARCHAR(255) (unique)
- `password_hash`: VARCHAR(255)
- `full_name`: VARCHAR(255)
- `department`: VARCHAR(255) (nullable)
- `reset_token`: VARCHAR(255) (nullable)
- `reset_token_expires_at`: TIMESTAMPTZ (nullable)
- `created_at`: TIMESTAMPTZ
- `updated_at`: TIMESTAMPTZ

### NixOS Configs Table
- `id`: UUID (primary key)
- `teacher_id`: UUID (foreign key)
- `filename`: VARCHAR(255)
- `content`: TEXT
- `file_size`: INTEGER
- `created_at`: TIMESTAMPTZ
- `updated_at`: TIMESTAMPTZ

## Development

### Running in Development Mode
```bash
cargo watch -x run
```

### Linting
```bash
cargo clippy
```

### Formatting
```bash
cargo fmt
```

## Production Considerations

1. **Environment Variables**: Use a secure secret management system
2. **Database**: Set up connection pooling appropriately
3. **Logging**: Configure structured logging for production
4. **CORS**: Restrict CORS origins to your frontend domain
5. **HTTPS**: Deploy behind a reverse proxy with TLS
6. **Email**: Implement actual email service for password resets
7. **Rate Limiting**: Add rate limiting middleware
8. **File Size Limits**: Add limits for configuration file sizes

## Frontend Integration

This backend is designed to work with a Deno + Svelte frontend. The API follows RESTful conventions and returns JSON responses.

### CORS Configuration
The current CORS configuration allows all origins for development. Update this in production:

```rust
let cors = CorsLayer::new()
    .allow_origin("https://your-frontend-domain.com".parse::<HeaderValue>().unwrap())
    .allow_methods([Method::GET, Method::POST, Method::PUT, Method::DELETE])
    .allow_headers(Any);
```

### Authentication Flow
1. Frontend calls `/api/auth/login`
2. Store the returned JWT token (localStorage/sessionStorage)
3. Include token in `Authorization: Bearer <token>` header for protected routes
4. Handle 401 responses by redirecting to login

## License

MIT

## Support

For issues or questions, please open an issue on the repository.
