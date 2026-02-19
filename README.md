# Yape Challenge - Transactions Service

This repository contains a Spring Boot 3 + Java 21 service using Hexagonal Architecture and DDD-style packaging.
It exposes CRUD endpoints for `transactions` and publishes/consumes Kafka events for antifraud / status flows.

## What I’m delivering (professional checklist)

- ✅ **API**: REST endpoints (GET/POST/PUT/DELETE)
- ✅ **Persistence**: PostgreSQL schema + JPA entities with optimistic locking (`@Version`)
- ✅ **Messaging**: Kafka topics for transaction lifecycle
- ✅ **Docs**: Clear setup + run steps, plus example cURLs and sample payloads
- ✅ **Reproducibility**: `docker-compose.yml` to run Postgres + Kafka locally

## Quick start (local)

### 1) Start dependencies (Postgres + Kafka)
```bash
docker compose up -d
```

This will:
- create database `yape`
- auto-run `db/schema.sql` on first initialization
- expose:
  - Postgres: `localhost:5432`
  - Kafka: `localhost:9092`
  - Kafka UI: `http://localhost:8081`

### 2) Configure the application
Example `application.yml` settings:
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/yape
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: validate   # IMPORTANT: validate against schema.sql
    properties:
      hibernate:
        format_sql: true
```

### 3) Run the app
```bash
./mvnw spring-boot:run
```

## Database

The DDL is in:
- `db/schema.sql`

Table:
- `public.transactions`

Key points:
- Primary key: `transaction_external_id (UUID)`
- Optimistic locking: `version (BIGINT)` with JPA `@Version`
- Basic business checks:
  - status in `PENDING|APPROVED|REJECTED`
  - value >= 0
  - debit != credit

## API examples

### Create
```bash
curl -X POST http://localhost:8080/transactions \
  -H "Content-Type: application/json" \
  -d '{
    "accountExternalIdDebit": "11111111-1111-1111-1111-111111111111",
    "accountExternalIdCredit": "22222222-2222-2222-2222-222222222222",
    "transferTypeId": 1,
    "value": 12.50
  }'
```

### Get
```bash
curl http://localhost:8080/transactions/<uuid>
```

### Update (only if status=PENDING)
```bash
curl -X PUT http://localhost:8080/transactions/<uuid> \
  -H "Content-Type: application/json" \
  -d '{
    "accountExternalIdDebit": "11111111-1111-1111-1111-111111111111",
    "accountExternalIdCredit": "22222222-2222-2222-2222-222222222222",
    "value": 15.75
  }'
```

### Delete
```bash
curl -X DELETE http://localhost:8080/transactions/<uuid>
```

> If you prefer returning a JSON confirmation instead of 204, adjust the controller to return a DTO/message.

## Extra deliverables to look senior (optional but recommended)

- **OpenAPI/Swagger** (`springdoc-openapi`) with examples for each endpoint
- **Postman collection** (export JSON) under `/docs/postman/`
- **Architecture diagram** (simple: ports/adapters + domain/application layers)
- **Tests**:
  - Unit tests for use cases
  - Integration test with Testcontainers (Postgres + Kafka)
- **GitHub**:
  - meaningful commit history
  - clean README
  - CI pipeline (GitHub Actions) running tests
