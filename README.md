# Q-Git-Revert-Env

A Flask-based API application demonstrating environment configuration, JWT authentication, and database integration with PostgreSQL and Redis support.

## Overview

This project showcases a modern Flask application with:
- Environment-based configuration using `.env` files
- JWT authentication for secure API access
- SQLAlchemy ORM for database management
- PostgreSQL database integration
- Redis caching support
- RESTful API endpoints
- Health check monitoring
- Version control at application level

## Features

- **Authentication**: JWT-based authentication for protected endpoints
- **Database Support**: PostgreSQL with SQLAlchemy ORM
- **Caching**: Redis integration for performance optimization
- **Health Monitoring**: Built-in health check endpoint
- **Environment Configuration**: Secure environment variable management
- **RESTful API**: Structured API endpoints with versioning
- **Testing**: Pytest integration for unit testing
- **Production Ready**: Gunicorn WSGI server support

## Tech Stack

- **Backend**: Flask 2.3.3
- **Database**: PostgreSQL (via psycopg2-binary)
- **ORM**: Flask-SQLAlchemy 3.1.1
- **Authentication**: Flask-JWT-Extended 4.5.3
- **Caching**: Redis 5.0.1
- **Server**: Gunicorn 21.2.0
- **Testing**: Pytest 7.4.3
- **Environment**: python-dotenv 1.0.0
- **Python**: 3.7+

## Project Structure

```
q-git-revert-env/
├── app.py                 # Main Flask application
├── requirements.txt       # Python dependencies (v7)
├── .env.example          # Example environment configuration
├── .gitignore            # Git ignore rules
├── .github/              # GitHub configuration
└── README.md             # This file
```

## Installation

### Prerequisites

- Python 3.7 or higher
- PostgreSQL database
- Redis server (optional, for caching)
- pip package manager

### Setup Instructions

1. **Clone the repository**:
```bash
git clone https://github.com/DS24F3004981/q-git-revert-env.git
cd q-git-revert-env
```

2. **Create a virtual environment**:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **Install dependencies**:
```bash
pip install -r requirements.txt
```

4. **Configure environment variables**:
```bash
cp .env.example .env
# Edit .env with your configuration
```

5. **Set required environment variables**:
```bash
export DATABASE_URL="postgresql://user:password@localhost:5432/dbname"
export JWT_SECRET="your-secret-key-here"
export PORT=5000
export API_KEY="your_api_key_here"
export DB_PASSWORD="your_db_password_here"
```

## Environment Configuration

### .env File Template

```
# Database Configuration
DATABASE_URL=postgresql://user:password@localhost:5432/database_name

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key-here

# Application Configuration
PORT=5000
FLASK_ENV=development
FLASK_DEBUG=True

# Redis Configuration (optional)
REDIS_URL=redis://localhost:6379/0

# API Keys
API_KEY=your_api_key_here
DB_PASSWORD=your_db_password_here
```

### Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| DATABASE_URL | PostgreSQL connection string | postgresql://user:pass@localhost:5432/db |
| JWT_SECRET | Secret key for JWT signing | your-secret-key |
| PORT | Port to run the application | 5000 |
| API_KEY | API authentication key | abc123xyz |
| DB_PASSWORD | Database password | secure_password |

## Database Models

### User Model

```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    created_at = db.Column(db.DateTime, default=db.func.now())
```

## API Endpoints

### Health Check

**GET** `/health`

Returns the application health status.

**Response**:
```json
{
  "status": "healthy",
  "version": "7.0.0"
}
```

### Authentication

**POST** `/api/v1/login`

Authenticate user and receive JWT token.

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response**:
```json
{
  "message": "Login endpoint",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

### List Users (Protected)

**GET** `/api/v1/users`

List all users (requires JWT authentication).

**Headers**:
```
Authorization: Bearer <access_token>
```

**Response**:
```json
{
  "users": [
    {
      "id": 1,
      "email": "user@example.com",
      "created_at": "2026-02-20T10:00:00"
    }
  ]
}
```

## Running the Application

### Development

```bash
# Run with Flask development server
python app.py

# Or with Flask CLI
export FLASK_APP=app.py
flask run
```

The application will be available at `http://localhost:5000`

### Production

```bash
# Run with Gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

## Testing

Run the test suite using pytest:

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run specific test file
pytest tests/test_auth.py

# Run with coverage
pytest --cov=.
```

## JWT Authentication

### How to Get a Token

1. Send a POST request to `/api/v1/login` with credentials
2. Receive an access token in response
3. Include the token in subsequent requests using the `Authorization` header

### Using the Token

```bash
curl -H "Authorization: Bearer <your_token>" \
  http://localhost:5000/api/v1/users
```

### Token Expiration

Tokens expire after the configured time. To refresh, send a new login request.

## Database Setup

### Create PostgreSQL Database

```bash
# Connect to PostgreSQL
psql -U postgres

# Create database
CREATE DATABASE q_git_revert_env;

# Create user
CREATE USER q_user WITH PASSWORD 'secure_password';

# Grant privileges
ALTER ROLE q_user SET client_encoding TO 'utf8';
ALTER ROLE q_user SET default_transaction_isolation TO 'read committed';
GRANT ALL PRIVILEGES ON DATABASE q_git_revert_env TO q_user;
```

### Initialize Database

```python
from app import app, db

with app.app_context():
    db.create_all()
    print("Database tables created successfully!")
```

## Redis Caching

### Setup Redis

```bash
# Install Redis
brew install redis  # macOS
# or
apt-get install redis-server  # Ubuntu

# Start Redis
redis-server
```

### Using Redis in Application

```python
from redis import Redis

redis_client = Redis.from_url(os.environ.get('REDIS_URL', 'redis://localhost:6379/0'))

# Cache example
def cache_user(user_id):
    user_data = redis_client.get(f'user:{user_id}')
    if not user_data:
        user = User.query.get(user_id)
        redis_client.setex(f'user:{user_id}', 3600, str(user))
    return user_data
```

## Error Handling

The application includes proper error handling for:

- Invalid credentials
- Missing JWT tokens
- Database connection errors
- Invalid input data
- Server errors

Example error response:

```json
{
  "error": "Invalid credentials",
  "status": 401
}
```

## Security Best Practices

1. **Environment Variables**: Never commit `.env` file
2. **JWT Secret**: Use a strong, random secret key
3. **Database Password**: Use secure passwords and connection strings
4. **HTTPS**: Use HTTPS in production
5. **Rate Limiting**: Implement rate limiting for API endpoints
6. **Input Validation**: Validate all user inputs
7. **SQL Injection**: Use SQLAlchemy ORM to prevent SQL injection
8. **CORS**: Configure CORS properly for your frontend

## Performance Optimization

1. **Database Indexing**: Index frequently queried fields
2. **Caching**: Use Redis for frequently accessed data
3. **Connection Pooling**: SQLAlchemy handles this automatically
4. **Pagination**: Implement pagination for list endpoints
5. **Query Optimization**: Use eager loading to avoid N+1 queries

## Monitoring

### Health Check

```bash
curl http://localhost:5000/health
```

### Application Logs

```bash
# View Gunicorn logs
journalctl -u q-git-revert-env -f
```

### Database Monitoring

```bash
# Connect to PostgreSQL and check table sizes
psql -d q_git_revert_env -c "\dt+"
```

## Deployment

### Docker Deployment

Create a `Dockerfile`:

```dockerfile
FROM python:3.9

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:5000", "app:app"]
```

Build and run:

```bash
docker build -t q-git-revert-env .
docker run -p 5000:5000 --env-file .env q-git-revert-env
```

### Heroku Deployment

```bash
# Create Procfile
echo "web: gunicorn app:app" > Procfile

# Deploy
git push heroku main
```

## Troubleshooting

### Database Connection Errors

```
Error: could not connect to server
```

Check that PostgreSQL is running and DATABASE_URL is correct.

### JWT Token Invalid

```
Error: Signature verification failed
```

Ensure JWT_SECRET matches on token creation and verification.

### Port Already in Use

```bash
# Use a different port
export PORT=5001
python app.py
```

### Redis Connection Failed

```
Error: Connection refused
```

Ensure Redis is running: `redis-server`

## Dependencies Updates

The project uses v7 of requirements with:
- Flask 2.3.3 (security updates)
- Flask-SQLAlchemy 3.1.1
- Flask-JWT-Extended 4.5.3
- Updated PostgreSQL driver

Check for updates:

```bash
pip list --outdated
```

## Contributing

1. Create a feature branch
2. Make your changes
3. Write/update tests
4. Commit with clear messages
5. Push and create a pull request

## Version History

- **v7.0.0**: Latest version with updated dependencies

## License

This project is open source and available under the MIT License.

## Author

Created by DS24F3004981

## Resources

- [Flask Documentation](https://flask.palletsprojects.com/)
- [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/)
- [Flask-JWT-Extended](https://flask-jwt-extended.readthedocs.io/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Redis Documentation](https://redis.io/documentation)
