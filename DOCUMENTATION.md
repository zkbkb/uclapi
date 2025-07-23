# UCL API - Comprehensive Project Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Technology Stack](#technology-stack)
4. [Project Structure](#project-structure)
5. [API Services](#api-services)
6. [Development Setup](#development-setup)
7. [Testing](#testing)
8. [Deployment](#deployment)
9. [API Documentation](#api-documentation)
10. [Contributing](#contributing)
11. [Security](#security)
12. [Maintenance](#maintenance)

## Project Overview

### What is UCL API?
UCL API is a **student-built** platform designed to provide developers with easy access to UCL's digital services through a comprehensive REST API. Originally created for **student developers** to improve the **student experience** at University College London, it has evolved to include end-user applications like UCL Assistant.

### Current Version
- **Version**: 1.4.11
- **License**: MIT License
- **Repository**: https://github.com/uclapi/uclapi

### Key Features
- OAuth 2.0 authentication with "Sign In With UCL"
- Multiple API services (Room Bookings, Timetables, Search, Resources, etc.)
- Rate limiting (10,000 requests per day per user)
- Comprehensive API documentation with OpenAPI 3.0 specification
- Real-time and cached data endpoints
- Student-focused applications ecosystem

## Architecture

### System Architecture
The UCL API follows a modern microservices-inspired architecture:

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Frontend      │────▶│   Backend API    │────▶│  UCL Systems    │
│   (Next.js)     │     │   (Django)       │     │  (Oracle, etc.) │
└─────────────────┘     └──────────────────┘     └─────────────────┘
        │                        │
        │                        │
        ▼                        ▼
┌─────────────────┐     ┌──────────────────┐
│   Static Files  │     │   Redis Cache    │
│   (S3/CDN)      │     │                  │
└─────────────────┘     └──────────────────┘
```

### Components
1. **Frontend (uclapi-frontend)**
   - Next.js 13.5.1 application
   - React 18.2.0 with TypeScript
   - Server-side rendering for performance
   - API documentation interface using OpenAPI Explorer

2. **Backend (backend/uclapi)**
   - Django 3.2.23 REST framework
   - Celery for asynchronous tasks
   - Redis for caching and session management
   - PostgreSQL for main database
   - Oracle database connections for UCL systems

3. **Infrastructure**
   - Docker containers for development and production
   - AWS services (ECR, S3)
   - Traefik for routing in production
   - GitHub Actions for CI/CD

## Technology Stack

### Frontend Technologies
- **Framework**: Next.js 13.5.1
- **UI Library**: React 18.2.0
- **Language**: TypeScript 5.0.4
- **Styling**: SCSS with include-media
- **State Management**: React hooks
- **Authentication**: NextAuth.js
- **API Documentation**: OpenAPI Explorer
- **Package Manager**: pnpm
- **Key Libraries**:
  - axios (HTTP client)
  - framer-motion (animations)
  - react-syntax-highlighter (code display)
  - rsuite (UI components)
  - Sentry (error tracking)

### Backend Technologies
- **Framework**: Django 3.2.23
- **API Framework**: Django REST Framework 3.14.0
- **Language**: Python 3.7+
- **Database**: PostgreSQL (primary), Oracle (UCL systems)
- **Cache**: Redis 4.6.0
- **Task Queue**: Celery 5.3.6
- **Web Server**: Gunicorn 21.2.0
- **Key Libraries**:
  - cx-Oracle (Oracle database connectivity)
  - django-cors-headers (CORS support)
  - boto3 (AWS SDK)
  - cryptography (security)
  - Sentry SDK (error tracking)

### DevOps & Infrastructure
- **Containerization**: Docker & Docker Compose
- **CI/CD**: GitHub Actions
- **Container Registry**: AWS ECR
- **Static Files**: AWS S3
- **Reverse Proxy**: Traefik (production), Django proxy (development)
- **Monitoring**: Sentry
- **Code Quality**: ESLint, Stylelint, Flake8, autopep8

## Project Structure

```
uclapi/
├── backend/
│   └── uclapi/
│       ├── common/          # Shared utilities and helpers
│       ├── dashboard/       # API dashboard and management
│       ├── libcal/         # Library calendar integration
│       ├── marketplace/    # App marketplace
│       ├── oauth/          # OAuth implementation
│       ├── resources/      # Desktop availability API
│       ├── roombookings/   # Room booking API
│       ├── search/         # Staff search API
│       ├── timetable/      # Timetable API
│       ├── uclapi/         # Main Django settings and config
│       └── workspaces/     # Study spaces API
├── uclapi-frontend/        # Next.js frontend application
├── docker/                 # Docker configuration files
├── .github/               # GitHub Actions workflows
└── docs/                  # Additional documentation

```

### Key Configuration Files
- `docker-compose.yml` - Production Docker configuration
- `docker-compose-dev.yml` - Development Docker configuration
- `uclapi.openapi.json` - OpenAPI 3.0 specification
- `backend/uclapi/.env.example` - Environment variables template
- `.github/workflows/workflow.yml` - Main CI/CD pipeline

## API Services

### 1. Room Bookings (`/roombookings/`)
- **Purpose**: Fetch details of room bookings within UCL
- **Endpoints**:
  - `/roombookings/rooms` - List all bookable rooms
  - `/roombookings/bookings` - Get room bookings
  - `/roombookings/equipment` - Get equipment in rooms
  - `/roombookings/freerooms` - Find available rooms
- **Data Freshness**: Updated every 20 minutes

### 2. Timetable (`/timetable/`)
- **Purpose**: Fetch timetables for students, modules, and courses
- **Endpoints**:
  - `/timetable/personal` - Personal timetable (OAuth required)
  - `/timetable/bymodule` - Timetable by module code
  - `/timetable/data/courses` - List of courses
  - `/timetable/data/modules` - List of modules
- **Data Freshness**: Updated every 20 minutes

### 3. OAuth (`/oauth/`)
- **Purpose**: Authenticate and authorize applications
- **Endpoints**:
  - `/oauth/authorise` - Authorization endpoint
  - `/oauth/token` - Token exchange
  - `/oauth/user/data` - Get user information
  - `/oauth/user/studentnumber` - Get student number (special scope)
- **Features**: Sign In With UCL button, scope-based permissions

### 4. Search (`/search/`)
- **Purpose**: Search for UCL staff members
- **Endpoints**:
  - `/search/people` - Search staff directory
- **Features**: Name and department search

### 5. Resources (`/resources/`)
- **Purpose**: Desktop PC availability
- **Endpoints**:
  - `/resources/desktops` - Get desktop availability by room
- **Data**: Real-time availability information

### 6. Workspaces (`/workspaces/`)
- **Purpose**: Library study space availability
- **Endpoints**:
  - `/workspaces/sensors` - Get sensor data
  - `/workspaces/maps` - Get workspace maps
  - `/workspaces/surveys` - Historical survey data
- **Data Freshness**: Updated every 2 minutes

### 7. LibCal (`/libcal/`)
- **Purpose**: Library seat reservation system
- **Endpoints**:
  - `/libcal/locations` - List bookable locations
  - `/libcal/available` - Check availability
  - `/libcal/reserve` - Make reservations
  - `/libcal/mybookings` - View user bookings

## Development Setup

### Prerequisites
- Docker and Docker Compose
- Python 3.7+
- Node.js 18+ and pnpm
- Git
- Redis (for local development without Docker)

### Quick Start (Docker)
```bash
# Clone repository
git clone https://github.com/uclapi/uclapi.git
cd uclapi

# Start development environment
docker-compose -f docker-compose-dev.yml up

# In a new terminal, setup frontend
cd uclapi-frontend
pnpm install
pnpm dev

# In another terminal, setup backend
cd backend/uclapi
cp .env.docker.example .env
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
python manage.py runserver
```

### Environment Configuration
The `.env` file should contain:
- Database credentials (PostgreSQL, Oracle)
- Redis configuration
- OAuth secrets
- AWS credentials (for production)
- Sentry DSN (for error tracking)
- API keys and secrets

### Database Setup
```bash
# Run migrations
python manage.py migrate

# Create superuser (optional)
python manage.py createsuperuser
```

## Testing

### Backend Testing
```bash
cd backend/uclapi
python manage.py test --settings=uclapi.settings_mocked
```

### Frontend Testing
```bash
cd uclapi-frontend
pnpm test  # Currently shows "TODO"
```

### Code Quality
- **Python**: Flake8 and autopep8 via pre-commit hooks
- **JavaScript/TypeScript**: ESLint via pre-commit hooks
- **SCSS**: Stylelint via pre-commit hooks
- **Automated checks**: HoundCI on pull requests

### Test Coverage
- Backend tests cover API endpoints, OAuth flow, data processing
- Uses mocked settings to avoid external dependencies
- Coverage reports via Codecov

## Deployment

### CI/CD Pipeline
1. **GitHub Actions Workflow** (`workflow.yml`):
   - Triggered on push to master or pull requests
   - Builds frontend and backend
   - Runs tests and linting
   - Deploys to staging/production

2. **Deployment Process**:
   - Build Docker images
   - Push to AWS ECR
   - Deploy using container orchestration
   - Update static files on S3/CDN

### Production Configuration
- Uses Gunicorn as WSGI server
- Traefik for reverse proxy and SSL
- PostgreSQL with connection pooling
- Redis for caching and sessions
- Celery workers for background tasks

### Monitoring
- Sentry for error tracking
- CloudWatch for infrastructure monitoring
- Application logs aggregated centrally

## API Documentation

### OpenAPI Specification
- Complete API documentation in `uclapi.openapi.json`
- OpenAPI 3.0 compliant
- Interactive documentation at `/docs`

### Authentication
1. **API Token**: For non-personal data
   - Generated in dashboard
   - Passed as `token` parameter

2. **OAuth Token**: For personal data
   - Obtained through OAuth flow
   - Used with `client_secret`

### Rate Limiting
- 10,000 requests per day per user
- Resets at midnight London time
- HTTP 429 with Retry-After header when exceeded

### Response Format
```json
{
  "ok": true,
  "data": {...},
  "_metadata": {
    "last_modified": "2024-01-15T10:30:00Z"
  }
}
```

## Contributing

### Development Workflow
1. Fork the repository
2. Create feature branch
3. Make changes following style guides
4. Write/update tests
5. Submit pull request

### Coding Standards
- **Python**: PEP8 with 4-space indentation
- **JavaScript**: JavaScript Standard Style
- **Commit Messages**: Follow `.github/.gitmessage` format

### Pull Request Process
1. Fill PR template
2. Pass automated checks
3. Code review by maintainers
4. Merge after approval

## Security

### Security Measures
- OAuth 2.0 for user authentication
- API token authentication
- CORS headers configuration
- Input validation and sanitization
- SQL injection prevention via ORM
- XSS protection in templates

### Vulnerability Reporting
- Email: isd.apiteam@ucl.ac.uk
- Do not create public issues for security vulnerabilities
- Hall of Fame for responsible disclosure

### Data Protection
- Personal data access requires OAuth consent
- Minimal data retention
- Compliance with GDPR
- Secure token storage

## Maintenance

### Regular Tasks
- Update dependencies (Dependabot)
- Monitor error rates (Sentry)
- Review and merge pull requests
- Update documentation
- Performance optimization

### Cache Management
- Room bookings: 20-minute cache
- Timetables: 20-minute cache
- Workspaces: 2-minute cache
- Manual cache refresh available

### Database Maintenance
- Regular backups
- Index optimization
- Query performance monitoring
- Connection pool management

### Monitoring Checklist
- [ ] API response times
- [ ] Error rates
- [ ] Database performance
- [ ] Cache hit rates
- [ ] Rate limit violations
- [ ] OAuth success rates

## Support and Contact

- **API Support**: isd.apiteam@ucl.ac.uk
- **Website**: https://uclapi.com
- **GitHub**: https://github.com/uclapi/uclapi
- **Staging Environment**: https://staging.ninja

---

*This documentation is maintained by the UCL API team. Last updated: Version 1.4.11*