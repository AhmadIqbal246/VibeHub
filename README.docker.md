# VibeHub Docker Setup

This guide will help you run VibeHub using Docker with full WebSocket support.

## Prerequisites

- Docker Desktop installed
- Docker Compose installed (comes with Docker Desktop)

## Quick Start

1. **Clone and navigate to the project directory**:
   ```bash
   cd F:\VibeHub
   ```

2. **Build and start all services**:
   ```bash
   docker-compose up --build
   ```

3. **Access the application**:
   - Frontend: http://localhost:5173
   - Backend API: http://localhost:8000
   - Database: localhost:5432
   - Redis: localhost:6379

## Services Included

- **Frontend**: React app running on Vite dev server (port 5173)
- **Backend**: Django with Daphne ASGI server for WebSocket support (port 8000)
- **Database**: PostgreSQL 15 (port 5432)
- **Redis**: For Django Channels and Celery (port 6379)
- **Celery**: Background task processing

## Development Workflow

### Starting Services
```bash
# Start all services
docker-compose up

# Start services in background
docker-compose up -d

# Rebuild and start (after code changes)
docker-compose up --build
```

### Stopping Services
```bash
# Stop all services
docker-compose down

# Stop and remove volumes (clean slate)
docker-compose down -v
```

### Individual Service Management
```bash
# Start only specific services
docker-compose up frontend backend

# View logs
docker-compose logs backend
docker-compose logs frontend
docker-compose logs -f  # follow logs

# Execute commands in containers
docker-compose exec backend python manage.py migrate
docker-compose exec backend python manage.py createsuperuser
docker-compose exec frontend npm install new-package
```

## Database Management

### Initial Setup
The database will be created automatically, but you need to run migrations:
```bash
docker-compose exec backend python manage.py migrate
docker-compose exec backend python manage.py createsuperuser
```

### Backup/Restore
```bash
# Backup
docker-compose exec db pg_dump -U ahmad1 chat_app > backup.sql

# Restore
docker-compose exec -T db psql -U ahmad1 chat_app < backup.sql
```

## Environment Variables

The docker-compose.yml includes all necessary environment variables. For production:

1. Copy `.env.docker` to `.env.docker.local`
2. Update the values in `.env.docker.local`
3. Reference the file in docker-compose.yml

## WebSocket Testing

Your WebSocket endpoints will be available at:
- `ws://localhost:8000/ws/chat/<conversation_id>/`
- `ws://localhost:8000/ws/conversations/`

## Troubleshooting

### Common Issues

1. **Port already in use**:
   ```bash
   # Find and kill processes using the ports
   netstat -ano | findstr :8000
   taskkill /PID <PID> /F
   ```

2. **Database connection issues**:
   ```bash
   # Check database logs
   docker-compose logs db
   
   # Restart database
   docker-compose restart db
   ```

3. **Frontend not updating**:
   ```bash
   # Rebuild frontend container
   docker-compose up --build frontend
   ```

4. **Clear everything and start fresh**:
   ```bash
   docker-compose down -v
   docker system prune -f
   docker-compose up --build
   ```

### Development Tips

- Code changes in both frontend and backend will be reflected immediately due to volume mounts
- The frontend supports hot reloading
- Backend will auto-restart on code changes if you add `--reload` to the Daphne command

### Production Deployment

For production, consider:
1. Using production-ready images
2. Setting `DEBUG=False`
3. Using proper secrets management
4. Setting up reverse proxy (nginx)
5. Using separate redis instances for channels and celery
6. Implementing proper logging and monitoring

## Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Frontend  │    │   Backend   │    │  Database   │
│   (React)   │────│  (Django)   │────│ (PostgreSQL)│
│   :5173     │    │   :8000     │    │   :5432     │
└─────────────┘    └─────────────┘    └─────────────┘
                           │
                    ┌─────────────┐    ┌─────────────┐
                    │    Redis    │    │   Celery    │
                    │   :6379     │────│   Worker    │
                    └─────────────┘    └─────────────┘
```

Your application is now fully Dockerized with WebSocket support!
