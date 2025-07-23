# UCL API Development Guide

## Introduction

Welcome to the UCL API development team! This guide will help you get started with contributing to the UCL API project. Whether you're fixing bugs, adding features, or improving documentation, this guide covers everything you need to know.

## Table of Contents

1. [Development Environment Setup](#development-environment-setup)
2. [Project Architecture Deep Dive](#project-architecture-deep-dive)
3. [Backend Development](#backend-development)
4. [Frontend Development](#frontend-development)
5. [Database Management](#database-management)
6. [Testing Guidelines](#testing-guidelines)
7. [Debugging Tips](#debugging-tips)
8. [Common Development Tasks](#common-development-tasks)
9. [Code Style Guide](#code-style-guide)
10. [Git Workflow](#git-workflow)

## Development Environment Setup

### Prerequisites

Ensure you have the following installed:
- **Docker** (20.10+) and **Docker Compose** (1.29+)
- **Python** (3.7+)
- **Node.js** (18+) and **pnpm** (7+)
- **Git**
- **Redis** (optional, if not using Docker)
- **PostgreSQL** (optional, if not using Docker)

### Initial Setup

1. **Fork and Clone the Repository**
   ```bash
   git clone https://github.com/YOUR_USERNAME/uclapi.git
   cd uclapi
   ```

2. **Set Up Git Hooks**
   ```bash
   git config commit.template ./.github/.gitmessage
   ```

3. **Start Docker Services**
   ```bash
   docker-compose -f docker-compose-dev.yml up -d
   ```

4. **Backend Setup**
   ```bash
   cd backend/uclapi
   cp .env.docker.example .env
   
   # Create virtual environment
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   
   # Install dependencies
   pip install -r requirements.txt
   
   # Run migrations
   python manage.py migrate
   
   # Create superuser (optional)
   python manage.py createsuperuser
   
   # Start development server
   python manage.py runserver
   ```

5. **Frontend Setup**
   ```bash
   cd uclapi-frontend
   pnpm install
   pnpm dev
   ```

### Environment Variables

Create a `.env` file in `backend/uclapi/` with the following structure:

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/uclapi
ORACLE_CONNECTION_STRING=//oracle.ucl.ac.uk:1521/PROD

# Redis
REDIS_URL=redis://localhost:6379

# Security
SECRET_KEY=your-secret-key
ALLOWED_HOSTS=localhost,127.0.0.1

# OAuth
OAUTH_CLIENT_ID=your-client-id
OAUTH_CLIENT_SECRET=your-client-secret

# AWS (for production)
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_STORAGE_BUCKET_NAME=uclapi-static

# Sentry (optional)
SENTRY_DSN=https://your-sentry-dsn
```

## Project Architecture Deep Dive

### Backend Architecture

The backend follows Django's MVT (Model-View-Template) pattern with REST framework:

```
backend/uclapi/
├── uclapi/              # Main Django project
│   ├── settings.py      # Django settings
│   ├── urls.py          # Root URL configuration
│   └── celery.py        # Celery configuration
├── common/              # Shared utilities
│   ├── decorators.py    # Custom decorators
│   ├── helpers.py       # Helper functions
│   └── middleware.py    # Custom middleware
├── dashboard/           # API dashboard app
├── oauth/               # OAuth implementation
├── roombookings/        # Room bookings API
├── timetable/           # Timetable API
├── search/              # Search API
├── resources/           # Resources API
├── workspaces/          # Workspaces API
└── libcal/              # LibCal integration
```

### Frontend Architecture

The frontend uses Next.js with the following structure:

```
uclapi-frontend/
├── pages/               # Next.js pages (routes)
│   ├── api/            # API routes
│   ├── docs/           # Documentation pages
│   └── dashboard/      # Dashboard pages
├── components/          # React components
│   ├── common/         # Shared components
│   ├── layout/         # Layout components
│   └── dashboard/      # Dashboard components
├── lib/                 # Utility functions
├── styles/              # SCSS styles
└── public/              # Static assets
```

### Data Flow

1. **Request Flow**:
   ```
   Client → Next.js → Django API → Database/Cache → Response
   ```

2. **Authentication Flow**:
   ```
   User → UCL SSO → OAuth Callback → Token Generation → Authenticated Requests
   ```

3. **Caching Strategy**:
   - Redis for session storage
   - Redis for API response caching
   - 20-minute cache for room bookings and timetables
   - 2-minute cache for workspace data

## Backend Development

### Creating a New API Endpoint

1. **Define the URL pattern** in `your_app/urls.py`:
   ```python
   from django.urls import path
   from . import views
   
   urlpatterns = [
       path('endpoint/', views.YourView.as_view(), name='your-endpoint'),
   ]
   ```

2. **Create the view** in `your_app/views.py`:
   ```python
   from rest_framework.views import APIView
   from rest_framework.response import Response
   from common.decorators import uclapi_protected_endpoint
   
   class YourView(APIView):
       @uclapi_protected_endpoint
       def get(self, request, *args, **kwargs):
           # Your logic here
           return Response({
               "ok": True,
               "data": {...}
           })
   ```

3. **Add models if needed** in `your_app/models.py`:
   ```python
   from django.db import models
   
   class YourModel(models.Model):
       name = models.CharField(max_length=100)
       created_at = models.DateTimeField(auto_now_add=True)
       
       class Meta:
           db_table = 'your_table_name'
   ```

4. **Create and run migrations**:
   ```bash
   python manage.py makemigrations your_app
   python manage.py migrate
   ```

### Working with Oracle Database

UCL systems use Oracle databases. Here's how to query them:

```python
from django.db import connections

def get_oracle_data():
    with connections['oracle'].cursor() as cursor:
        cursor.execute("""
            SELECT column1, column2
            FROM ucl_table
            WHERE condition = :param
        """, {'param': value})
        
        columns = [col[0] for col in cursor.description]
        return [dict(zip(columns, row)) for row in cursor.fetchall()]
```

### Implementing Caching

Use Redis caching for expensive operations:

```python
from django.core.cache import cache
from datetime import timedelta

def get_cached_data(key):
    # Try to get from cache
    data = cache.get(key)
    if data is not None:
        return data
    
    # Fetch fresh data
    data = expensive_operation()
    
    # Store in cache
    cache.set(key, data, timeout=timedelta(minutes=20).total_seconds())
    return data
```

### Adding Celery Tasks

For background processing:

```python
from celery import shared_task
from datetime import datetime

@shared_task
def process_data_async(data_id):
    # Long-running task
    result = process_data(data_id)
    
    # Store result
    cache.set(f'result_{data_id}', result)
    return result
```

## Frontend Development

### Creating a New Page

1. **Create the page component** in `pages/your-page.tsx`:
   ```tsx
   import { NextPage } from 'next'
   import Layout from '../components/layout/Layout'
   
   const YourPage: NextPage = () => {
     return (
       <Layout>
         <h1>Your Page Title</h1>
         {/* Your content */}
       </Layout>
     )
   }
   
   export default YourPage
   ```

2. **Add API integration** using SWR or fetch:
   ```tsx
   import useSWR from 'swr'
   
   const fetcher = (url: string) => fetch(url).then(res => res.json())
   
   const YourComponent = () => {
     const { data, error } = useSWR('/api/endpoint', fetcher)
     
     if (error) return <div>Error loading data</div>
     if (!data) return <div>Loading...</div>
     
     return <div>{/* Render your data */}</div>
   }
   ```

3. **Style your component** in `styles/YourComponent.module.scss`:
   ```scss
   @import 'include-media';
   
   .container {
     padding: 2rem;
     
     @include media('>=tablet') {
       padding: 3rem;
     }
   }
   ```

### Working with Authentication

Use NextAuth for authentication:

```tsx
import { useSession } from 'next-auth/react'

const ProtectedComponent = () => {
  const { data: session, status } = useSession()
  
  if (status === 'loading') return <p>Loading...</p>
  if (status === 'unauthenticated') return <p>Please sign in</p>
  
  return <div>Welcome {session.user.name}</div>
}
```

## Database Management

### Running Migrations

```bash
# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Rollback migrations
python manage.py migrate app_name migration_name
```

### Database Shell Access

```bash
# PostgreSQL
python manage.py dbshell

# Direct PostgreSQL access
psql -h localhost -U postgres -d uclapi
```

### Data Fixtures

Create test data fixtures:

```bash
# Export data
python manage.py dumpdata app_name.ModelName > fixtures/data.json

# Load data
python manage.py loaddata fixtures/data.json
```

## Testing Guidelines

### Backend Testing

1. **Unit Tests**:
   ```python
   from django.test import TestCase
   from django.urls import reverse
   from rest_framework.test import APIClient
   
   class YourAPITestCase(TestCase):
       def setUp(self):
           self.client = APIClient()
           
       def test_endpoint_success(self):
           response = self.client.get(reverse('your-endpoint'))
           self.assertEqual(response.status_code, 200)
           self.assertTrue(response.json()['ok'])
   ```

2. **Run Tests**:
   ```bash
   # Run all tests
   python manage.py test --settings=uclapi.settings_mocked
   
   # Run specific app tests
   python manage.py test roombookings --settings=uclapi.settings_mocked
   
   # Run with coverage
   coverage run --source='.' manage.py test
   coverage report
   ```

### Frontend Testing

```tsx
import { render, screen } from '@testing-library/react'
import YourComponent from './YourComponent'

describe('YourComponent', () => {
  it('renders correctly', () => {
    render(<YourComponent />)
    expect(screen.getByText('Expected Text')).toBeInTheDocument()
  })
})
```

## Debugging Tips

### Backend Debugging

1. **Django Debug Toolbar**:
   ```python
   # In development settings
   if DEBUG:
       INSTALLED_APPS += ['debug_toolbar']
       MIDDLEWARE += ['debug_toolbar.middleware.DebugToolbarMiddleware']
   ```

2. **Logging**:
   ```python
   import logging
   
   logger = logging.getLogger(__name__)
   
   def your_view(request):
       logger.debug(f"Request data: {request.data}")
       try:
           # Your code
       except Exception as e:
           logger.error(f"Error occurred: {str(e)}", exc_info=True)
   ```

3. **PDB Debugging**:
   ```python
   import pdb
   
   def problematic_function():
       pdb.set_trace()  # Debugger will stop here
       # Your code
   ```

### Frontend Debugging

1. **React Developer Tools**: Install browser extension
2. **Console Logging**:
   ```tsx
   console.log('Component props:', props)
   console.table(data)
   ```
3. **Network Tab**: Monitor API calls in browser DevTools

## Common Development Tasks

### Adding a New Dependency

**Backend**:
```bash
pip install package-name
pip freeze > requirements.txt
```

**Frontend**:
```bash
pnpm add package-name
```

### Updating the OpenAPI Specification

1. Update the specification in `uclapi.openapi.json`
2. Validate using an OpenAPI validator
3. Update both frontend and root copies

### Running Celery Workers

```bash
# Start worker
celery -A uclapi worker -l info

# Start beat scheduler
celery -A uclapi beat -l info
```

### Database Backups

```bash
# Backup
pg_dump -h localhost -U postgres uclapi > backup.sql

# Restore
psql -h localhost -U postgres uclapi < backup.sql
```

## Code Style Guide

### Python (PEP 8)

```python
# Good
def calculate_average(numbers: List[float]) -> float:
    """Calculate the average of a list of numbers."""
    if not numbers:
        return 0.0
    return sum(numbers) / len(numbers)

# Bad
def calc(n):
    if not n: return 0
    return sum(n)/len(n)
```

### TypeScript/JavaScript

```typescript
// Good
interface UserData {
  id: string
  name: string
  email: string
}

const fetchUserData = async (userId: string): Promise<UserData> => {
  const response = await fetch(`/api/users/${userId}`)
  return response.json()
}

// Bad
const fetchData = async (id) => {
  const res = await fetch('/api/users/' + id)
  return res.json()
}
```

### SCSS

```scss
// Good
.component {
  &__element {
    color: $primary-color;
    
    &--modifier {
      font-weight: bold;
    }
  }
}

// Bad
.component__element {
  color: #007bff;
}
.component__element--modifier {
  font-weight: bold;
}
```

## Git Workflow

### Branch Naming

- Feature: `feature/add-new-endpoint`
- Bug fix: `fix/oauth-token-expiry`
- Documentation: `docs/update-api-guide`
- Refactor: `refactor/optimize-queries`

### Commit Messages

Follow the format in `.github/.gitmessage`:

```
area subarea: message

Detailed explanation of the change
```

Examples:
- `backend roombookings: add capacity filter to rooms endpoint`
- `frontend dashboard: fix responsive layout on mobile`
- `docs api: update authentication examples`

### Pull Request Process

1. **Create feature branch**:
   ```bash
   git checkout -b feature/your-feature
   ```

2. **Make changes and commit**:
   ```bash
   git add .
   git commit
   ```

3. **Push to your fork**:
   ```bash
   git push origin feature/your-feature
   ```

4. **Create pull request**:
   - Fill out the PR template
   - Link related issues
   - Add appropriate labels
   - Request reviews

5. **Address review feedback**:
   ```bash
   git add .
   git commit -m "address review comments"
   git push origin feature/your-feature
   ```

### Keeping Your Fork Updated

```bash
# Add upstream remote
git remote add upstream https://github.com/uclapi/uclapi.git

# Fetch upstream changes
git fetch upstream

# Merge or rebase
git checkout master
git merge upstream/master

# Update your fork
git push origin master
```

## Troubleshooting

### Common Issues

1. **Docker containers not starting**:
   ```bash
   docker-compose down
   docker-compose -f docker-compose-dev.yml up --build
   ```

2. **Database connection errors**:
   - Check `.env` file configuration
   - Ensure PostgreSQL is running
   - Verify credentials

3. **Frontend build errors**:
   ```bash
   rm -rf node_modules pnpm-lock.yaml
   pnpm install
   ```

4. **Migration conflicts**:
   ```bash
   python manage.py migrate --fake app_name
   python manage.py makemigrations --merge
   ```

### Getting Help

- Check existing issues on GitHub
- Ask in the development chat
- Email: isd.apiteam@ucl.ac.uk

---

*Happy coding! Remember to test your changes and follow the style guides.*