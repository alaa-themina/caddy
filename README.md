# Caddy Reverse Proxy Configuration

This directory contains the Caddy reverse proxy configuration for the Themina project. Caddy serves as a single entry point that routes requests to the appropriate services.

## Architecture

```
themina.local
├── /api/*          → Backend (Laravel) on gr_web:80
├── /dashboard/*    → Admin Frontend (Next.js) on admin-frontend:4000
└── /*              → Core Frontend (Next.js) on core-frontend:3000
```

## Setup Instructions

### 1. Configure Hosts File

Add the following to your `/etc/hosts` file:

```bash
127.0.0.1 themina.local
```

### 2. Configure Environment Variables

Each project has its own `.env` file with the `BASE_DOMAIN` variable:

- `/caddy/.env`
- `/core-backend/.env` (also add FRONTEND_URL and ADMIN_FRONTEND_URL)
- `/core-frontend/.env`
- `/admin-frontend/.env`

All `.env` files should contain:
```bash
BASE_DOMAIN=themina.local
```

For the frontend projects, you can also set:
```bash
USE_LOCALHOST=false  # Set to true to bypass Caddy during development
```

### 3. Start the Services

Start services in the following order:

```bash
# 1. Start the backend first (creates the network)
cd /home/medo/work/themina/core-backend
docker-compose up -d

# 2. Start the core frontend
cd /home/medo/work/themina/core-frontend
docker-compose up -d

# 3. Start the admin frontend
cd /home/medo/work/themina/admin-frontend
docker-compose up -d

# 4. Start Caddy
cd /home/medo/work/themina/caddy
docker-compose up -d
```

### 4. Access the Applications

Once all services are running:

- **Core Frontend**: http://themina.local
- **Admin Dashboard**: http://themina.local/dashboard
- **Backend API**: http://themina.local/api

## Configuration Files

### Caddyfile

The main routing configuration. It uses the `BASE_DOMAIN` environment variable and forwards requests to the appropriate service.

### docker-compose.yml

Defines the Caddy service and connects it to the shared `core-backend_ksp` network.

## Development Mode

If you want to bypass Caddy and connect directly to the backend during development:

1. Set `USE_LOCALHOST=true` in the frontend `.env` files
2. Access services directly:
   - Core Frontend: http://localhost:3000
   - Admin Frontend: http://localhost:4000
   - Backend: http://localhost:8001

## Production Deployment

For production deployment:

1. Update `BASE_DOMAIN` in all `.env` files:
   ```bash
   BASE_DOMAIN=themina.sa
   ```

2. Update backend `.env`:
   ```bash
   BASE_DOMAIN=themina.sa
   FRONTEND_URL=https://themina.sa
   ADMIN_FRONTEND_URL=https://themina.sa/dashboard
   APP_ENV=production
   ```

3. Caddy will automatically obtain SSL certificates from Let's Encrypt

## Troubleshooting

### Services can't communicate

Ensure all services are on the same Docker network:
```bash
docker network inspect core-backend_ksp
```

### Caddy can't find services

Check that container names match the Caddyfile:
```bash
docker ps | grep -E "gr_web|core-frontend|admin-frontend"
```

### CORS errors

Check the backend's `config/cors.php` to ensure the frontend URLs are allowed.

### Admin dashboard 404 errors

Ensure the admin frontend is running and has `basePath: '/dashboard'` in its `next.config.js`.

## Logs

View Caddy logs:
```bash
docker logs -f themina_caddy
```

## Network Information

All services must be connected to the `core-backend_ksp` network:
- `gr_web` (backend nginx)
- `gr_app` (backend PHP)
- `core-frontend` (core frontend)
- `admin-frontend` (admin frontend)
- `themina_caddy` (Caddy reverse proxy)


