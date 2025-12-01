# Bookstore Microservices Platform

A microservices-based platform for managing an online bookstore, built with Go using gRPC, MongoDB, Redis, and NATS.

## Architecture

The project consists of the following microservices:

- **API Gateway** - single entry point for all HTTP requests (port 8080)
- **Book Service** - book catalog management (gRPC port 50051)
- **User Service** - user management (gRPC port 50053)
- **Order Service** - order processing (gRPC port 50052)
- **User Library Service** - user library management (gRPC port 50053)
- **Exchange Service** - book exchange between users (gRPC port 50054)
- **Notification Service** - sending notifications to users (gRPC port 50055)

## Technology Stack

- **Language**: Go 1.24.3
- **Database**: MongoDB 5.0
- **Caching**: Redis
- **Inter-service Communication**: gRPC, Protocol Buffers
- **Message System**: NATS 2.9
- **API Gateway**: Gin Framework
- **Containerization**: Docker, Docker Compose
- **Monitoring**: Prometheus

## Requirements

- Docker 20.10+
- Docker Compose 1.29+
- Go 1.24+ (for local development)
- Make (optional)

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/OshakbayAigerim/read_space.git
cd read_space
```

### 2. Run with Docker Compose

```bash
docker-compose up --build
```

This will start all services and dependencies. The API Gateway will be available at `http://localhost:8080`

### 3. Verify Services

```bash
# Check MongoDB availability
docker-compose exec mongo mongo --eval "db.adminCommand('ping')"

# Check NATS availability
docker-compose exec nats nats ping
```

## Project Structure

```
.
├── api_gateway/              # HTTP API Gateway
│   ├── cmd/
│   │   └── main.go
│   └── Dockerfile
├── book_service/             # Book management service
│   ├── cmd/
│   ├── internal/
│   │   ├── cache/           # Redis caching
│   │   ├── config/          # Configuration
│   │   ├── domain/          # Domain models
│   │   ├── handler/         # gRPC handlers
│   │   ├── repository/      # Database layer
│   │   └── usecase/         # Business logic
│   ├── proto/               # Protocol Buffers definitions
│   └── Dockerfile
├── user_service/             # User management service
│   ├── cmd/
│   ├── internal/
│   │   ├── migration/       # Database migrations
│   │   └── ...
│   ├── proto/
│   └── Dockerfile
├── order_service/            # Order processing service
├── user_library_service/     # User library service
├── exchange_service/         # Book exchange service
├── notification_service/     # Notification service
├── docker-compose.yml        # Docker Compose configuration
├── go.mod                    # Go modules
└── go.sum
```

## API Endpoints

### Books
- `GET /books/*` - retrieve book information
- `POST /books/*` - create/update a book
- `DELETE /books/*` - delete a book

### Users
- `GET /users/*` - retrieve user information
- `POST /users/*` - create/update a user
- `DELETE /users/*` - delete a user

### Orders
- `GET /orders/*` - retrieve order information
- `POST /orders/*` - create an order
- `PUT /orders/*` - update an order

### Libraries
- `GET /libraries/*` - retrieve user library
- `POST /libraries/*` - add a book to library
- `DELETE /libraries/*` - remove a book from library

### Exchange
- `GET /exchange/*` - retrieve available exchanges
- `POST /exchange/*` - create an exchange request
- `PUT /exchange/*` - accept/reject an exchange

### Notifications
- `GET /notifications/*` - retrieve notifications
- `POST /notifications/*` - send notifications

## Development

### Running a Service Locally

```bash
# Install dependencies
go mod download

# Run a specific service (e.g., book_service)
cd book_service
go run cmd/main.go
```

### Environment Variables

Each service uses the following environment variables:

```env
MONGO_URI=mongodb://mongo:27017/readspace
NATS_URL=nats://nats:4222
REDIS_URL=redis://redis:6379
```

### Generating gRPC Code

```bash
# Install protoc and plugins
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

# Generate code for a service
cd book_service
protoc --go_out=. --go-grpc_out=. proto/*.proto
```

## Monitoring and Logging

### Prometheus Metrics

Services export Prometheus metrics at the following endpoints:
- Book Service: `http://localhost:50051/metrics`
- User Service: `http://localhost:50053/metrics`
- Order Service: `http://localhost:50052/metrics`

### Logs

View service logs:

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f book_service
docker-compose logs -f api_gateway
```

## Testing

```bash
# Run tests for all services
go test ./...

# Run tests for a specific service
cd exchange_service
go test ./internal/usecase/...
```

## Shutdown and Cleanup

```bash
# Stop all services
docker-compose down

# Stop with volume removal (database will be cleared)
docker-compose down -v

# Remove all images
docker-compose down --rmi all
```

## Architectural Features

### Clean Architecture

Each service follows clean architecture principles:
- **Domain** - domain models and interfaces
- **UseCase** - business logic
- **Repository** - data storage operations
- **Handler** - incoming request handlers

### Caching

Book Service uses Redis for caching frequently requested data, reducing load on MongoDB.

### Asynchronous Communication

Services use NATS for asynchronous event exchange, ensuring loose coupling between components.

## Scaling

### Horizontal Scaling

```bash
# Run multiple instances of a service
docker-compose up --scale book_service=3
```

### Vertical Scaling

Configure resources in `docker-compose.yml`:

```yaml
book_service:
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 2G
      reservations:
        cpus: '1'
        memory: 1G
```

## Troubleshooting

### Service Won't Start

1. Check logs: `docker-compose logs <service_name>`
2. Ensure ports are not in use: `netstat -an | grep <port>`
3. Verify dependency availability (MongoDB, NATS)

### Database Connection Issues

```bash
# Check MongoDB status
docker-compose exec mongo mongo --eval "db.runCommand({ connectionStatus: 1 })"

# Recreate MongoDB container
docker-compose up -d --force-recreate mongo
```

### gRPC Errors

1. Ensure proto files are up to date
2. Regenerate gRPC code
3. Check dependency versions in `go.mod`

## Security

- All inter-service connections occur within Docker network
- MongoDB has no external access (port 27017 is open for development only)
- TLS for gRPC is recommended in production
- Add authentication to API Gateway

## Performance

- Redis caching for frequently requested data
- Connection pooling for MongoDB
- gRPC for efficient inter-service communication
- Asynchronous event processing via NATS


## Additional Resources

- [gRPC Documentation](https://grpc.io/docs/)
- [MongoDB Go Driver](https://pkg.go.dev/go.mongodb.org/mongo-driver)
- [NATS Documentation](https://docs.nats.io/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Gin Framework](https://gin-gonic.com/docs/)
