# Quick Start Guide

Get the distributed e-commerce system running in 5 minutes!

## Prerequisites

- Docker & Docker Compose installed
- Git
- (Optional) Go 1.21+ for local development

## Option 1: Quick Start with Docker (Recommended)

### 1. Start Everything

```bash
# Clone the repository
git clone https://github.com/seemasultana7362/E-Commerce-Distributed-System-Architecture.git
cd E-Commerce-Distributed-System-Architecture

# Start  all services 
make dev
```
   
That's it! The system will start:
- ✅ PostgreSQL Auth Database
- ✅ PostgreSQL Product Database
- ✅ Auth Service (Port 8001)
- ✅ Product Service (Port 8002)
- ✅ NGINX API Gateway (Port 80)

### 2. Verify Services are Running

```bash
# Check health
make health-check

# Or manually
curl http://localhost:8001/health
curl http://localhost:8002/health
```

### 3. Test the APIs

**Register a user:**
```bash
curl -X POST http://localhost/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "demo@example.com",
    "password": "Demo@123456",
    "first_name": "Demo",
    "last_name": "User"
  }'
```

**Login:**
```bash
curl -X POST http://localhost/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "demo@example.com",
    "password": "Demo@123456"
  }'
```

**List products:**
```bash
curl http://localhost/api/v1/products?page=1&page_size=10
```

**Get categories:**
```bash
curl http://localhost/api/v1/categories
```

### 4. Stop Services

```bash
make down
```

### 5. View Logs

```bash
# All services
make logs

# Specific service
make logs-auth
make logs-product
make logs-nginx
```

## Option 2: Local Development (No Docker)

### 1. Prerequisites

```bash
# Install PostgreSQL
brew install postgresql  # macOS
# or
sudo apt-get install postgresql  # Ubuntu

# Install Go 1.21+
brew install go  # macOS
# or visit https://golang.org/doc/install
```

### 2. Start Databases

```bash
# Start PostgreSQL
brew services start postgresql  # macOS
# or
sudo systemctl start postgresql  # Ubuntu

# Create databases
createdb ecommerce_auth -U postgres
createdb ecommerce_product -U postgres
```

### 3. Run Services

**Terminal 1 - Auth Service:**
```bash
make run-auth
```

**Terminal 2 - Product Service:**
```bash
make run-product
```

### 4. Test APIs

```bash
# Same API calls as above
curl -X POST http://localhost:8001/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{...}'
```

## API Endpoints

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/auth/register` | Register new user |
| POST | `/api/v1/auth/login` | Login user |
| POST | `/api/v1/auth/refresh` | Refresh access token |
| GET | `/api/v1/auth/me` | Get current user* |
| POST | `/api/v1/auth/logout` | Logout user* |

*Requires Authorization header with Bearer token

### Products

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v1/products` | List products (paginated) |
| GET | `/api/v1/products/{id}` | Get product by ID |
| GET | `/api/v1/categories` | Get all categories |
| POST | `/api/v1/products` | Create product (admin only)* |
| PUT | `/api/v1/products/{id}` | Update product (admin only)* |
| DELETE | `/api/v1/products/{id}` | Delete product (admin only)* |

*Requires Authorization header with Bearer token from admin account

## Database Access

### Connect to Auth Database

```bash
psql -U postgres -d ecommerce_auth -h localhost
```

### Connect to Product Database

```bash
psql -U postgres -d ecommerce_product -h localhost -p 5433
```

Default credentials:
- Username: `postgres`
- Password: `postgres`

## Environment Variables

Create `.env` file to override defaults:

```bash
# Auth Service
DATABASE_URL=postgres://postgres:postgres@localhost:5432/ecommerce_auth?sslmode=disable
JWT_SECRET=your-super-secret-key-change-in-production
PORT=8001

# Product Service
DATABASE_URL=postgres://postgres:postgres@localhost:5433/ecommerce_product?sslmode=disable
JWT_SECRET=your-super-secret-key-change-in-production
PORT=8002
```

## Project Structure

```
ecommerce-system/
├── services/
│   ├── auth/               # Authentication service
│   ├── product/            # Product catalog service
│   └── [other services]
├── shared/                 # Shared code
│   ├── models/            # Data models
│   ├── middleware/        # HTTP middleware
│   └── utils/             # Utility functions
├── deployments/
│   ├── docker/            # Docker configs
│   └── k8s/               # Kubernetes configs
├── docs/
│   └── PHASE_1_IMPLEMENTATION.md
├── Makefile               # Development commands
├── docker-compose.yml     # Docker Compose setup
└── go.mod                 # Go module file
```

## Common Commands

```bash
# Development
make dev              # Start everything
make build            # Build Docker images
make up               # Start services
make down             # Stop services
make clean            # Remove containers & volumes

# Local development
make build-auth       # Build auth service
make run-auth         # Run auth service
make build-product    # Build product service
make run-product      # Run product service

# Testing
make test-auth-register
make test-auth-login
make test-products-list
make test-categories

# Monitoring
make health-check
make logs
make logs-auth
make logs-product
make logs-nginx
```

## Troubleshooting

### Services won't start

```bash
# Check if ports are in use
lsof -i :80
lsof -i :8001
lsof -i :8002

# Or with netstat
netstat -tulpn | grep LISTEN

# Stop and clean up
make clean
make dev
```

### Database connection errors

```bash
# Verify containers are running
docker ps

# Check PostgreSQL status
docker logs ecommerce-postgres-auth

# Manually test connection
psql -U postgres -h localhost -d ecommerce_auth
```

### JWT token errors

```bash
# Token expired? Use refresh endpoint
curl -X POST http://localhost/api/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{"refresh_token": "your-refresh-token"}'
```

### CORS errors

- CORS is enabled on all services
- Check NGINX configuration: `deployments/docker/nginx.conf`
- Verify `Access-Control-Allow-Origin` header

## Production Deployment

For production, follow these steps:

1. **Change JWT Secret**
   ```bash
   # Generate strong secret
   openssl rand -base64 32
   
   # Update in environment
   export JWT_SECRET="<generated-secret>"
   ```

2. **Use environment-specific configs**
   ```bash
   docker-compose -f docker-compose.prod.yml up -d
   ```

3. **Enable HTTPS/SSL**
   - Update NGINX configuration
   - Use Let's Encrypt certificates

4. **Database backups**
   ```bash
   pg_dump -U postgres ecommerce_auth > auth_backup.sql
   pg_dump -U postgres ecommerce_product > product_backup.sql
   ```

5. **Monitor services**
   - Set up health check monitoring
   - Configure log aggregation
   - Set up alerting

## Next Steps

After getting Phase 1 running:

1. **Read the detailed documentation:**
   ```bash
   cat docs/PHASE_1_IMPLEMENTATION.md
   ```

2. **Understand the architecture:**
   - Review README.md
   - Study the architecture diagram

3. **Implement Phase 2:**
   - Order Service
   - Inventory Service
   - Event-driven communication

4. **Add features:**
   - Redis caching
   - Database replication
   - Kafka event bus
   - Advanced observability

## Support & Issues

- Check existing issues on GitHub
- Review logs: `make logs`
- Verify configuration in `docker-compose.yml`
- Consult `docs/PHASE_1_IMPLEMENTATION.md` for detailed info

## Performance Testing

Once running, test performance:

```bash
# Simple load test with Apache Bench
ab -n 1000 -c 10 http://localhost/api/v1/products

# Or with hey
go install github.com/rakyll/hey@latest
hey -n 1000 -c 10 http://localhost/api/v1/products
```

## Learning Path

1. **Week 1**: Get Phase 1 running and understand the structure
2. **Week 2**: Study each service's implementation
3. **Week 3**: Implement Phase 2 (Order & Inventory)
4. **Week 4**: Add event-driven communication
5. **Week 5+**: Implement advanced patterns (caching, scaling, etc.)

## Resources

- [Go Best Practices](https://golang.org/doc/effective_go)
- [12 Factor App](https://12factor.net/)
- [Microservices Architecture](https://microservices.io/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Documentation](https://docs.docker.com/)

---

**Happy Coding! 🚀**

For detailed documentation, see [PHASE_1_IMPLEMENTATION.md](./docs/PHASE_1_IMPLEMENTATION.md)
