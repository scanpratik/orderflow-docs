# 📋 SDD Spec — Order Service (OrderFlow v1)

> **Methodology**: Spec-Driven Development — review and approve this before any code is written.  
> **Project**: OrderFlow · **Service**: order-service · **Phase**: Week 2–4  
> **Compute**: AWS Lambda (via Spring Cloud Function)

---

## 1. What Are We Building?

A **production-grade REST API** for managing orders, deployed as an **AWS Lambda function** behind API Gateway.  
It demonstrates: clean layered architecture, JPA persistence, input validation, JWT auth, global error handling, and OpenAPI docs — all running serverless on Lambda.

---

## 2. Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Java 21 (virtual threads enabled) |
| Framework | Spring Boot 3.4.x + **Spring Cloud Function** |
| Build | Gradle (Kotlin DSL) |
| Database | PostgreSQL — RDS Aurora Serverless v2 (via Lambda VPC) |
| ORM | Spring Data JPA + Hibernate |
| Auth | Spring Security + JWT (stateless) |
| Docs | SpringDoc OpenAPI (Swagger UI) |
| Testing | JUnit 5 + Mockito + Testcontainers |
| Deployment | **AWS Lambda** + API Gateway (HTTP API) via AWS CDK |

---

## 3. API Contract

**Base URL**: `/api/v1/orders` (via API Gateway)  
**Auth**: Bearer token (JWT) required on all endpoints except health check.

### Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| `GET` | `/api/v1/orders` | List all orders (paginated) | ✅ |
| `GET` | `/api/v1/orders/{id}` | Get order by ID | ✅ |
| `POST` | `/api/v1/orders` | Create a new order | ✅ |
| `PUT` | `/api/v1/orders/{id}/status` | Update order status | ✅ |
| `DELETE` | `/api/v1/orders/{id}` | Cancel an order | ✅ |
| `GET` | `/actuator/health` | Health check | ❌ |

### Request / Response Schemas

**POST `/api/v1/orders`** — Create Order
```json
// Request
{
  "customerId": "cust-123",
  "items": [
    { "productId": "prod-456", "quantity": 2, "unitPrice": 29.99 }
  ]
}

// Response 201 Created
{
  "id": "ord-789",
  "customerId": "cust-123",
  "status": "PENDING",
  "totalAmount": 59.98,
  "items": [...],
  "createdAt": "2026-02-23T00:00:00Z"
}
```

**PUT `/api/v1/orders/{id}/status`** — Update Status
```json
// Request
{ "status": "CONFIRMED" }

// Response 200 OK
{ "id": "ord-789", "status": "CONFIRMED", "updatedAt": "..." }
```

**Error Response (all endpoints)**
```json
{ "timestamp": "...", "status": 404, "error": "Order not found", "path": "/api/v1/orders/ord-999" }
```

---

## 4. Data Model

### `orders` table
| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | Auto-generated |
| `customer_id` | VARCHAR(100) | Required |
| `status` | ENUM | PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED |
| `total_amount` | DECIMAL(10,2) | Computed from items |
| `created_at` | TIMESTAMP | Auto |
| `updated_at` | TIMESTAMP | Auto |

### `order_items` table
| Column | Type | Notes |
|--------|------|-------|
| `id` | UUID (PK) | Auto-generated |
| `order_id` | UUID (FK) | References `orders.id` |
| `product_id` | VARCHAR(100) | Required |
| `quantity` | INT | Min: 1 |
| `unit_price` | DECIMAL(10,2) | Min: 0.01 |

---

## 5. Business Rules

- An order must have **at least 1 item**
- `quantity` must be ≥ 1, `unit_price` must be > 0
- `totalAmount` is auto-calculated (sum of quantity × unitPrice)
- Only `PENDING` orders can be `CANCELLED`
- Status transitions: `PENDING → CONFIRMED → SHIPPED → DELIVERED`
- `CANCELLED` is a terminal state

---

## 6. Acceptance Criteria

- [ ] `POST /orders` returns `201` with a valid order + UUID
- [ ] `GET /orders` returns paginated list (default page size: 20)
- [ ] `GET /orders/{id}` returns `404` with error body if not found
- [ ] Invalid request body returns `400` with field-level validation errors
- [ ] Missing/invalid JWT returns `401`
- [ ] Cancelling a non-PENDING order returns `409 Conflict`
- [ ] Swagger UI accessible at `/swagger-ui.html` (local) and via API GW stage URL
- [ ] All layers have ≥ 80% test coverage
- [ ] Lambda cold start < 3s (SnapStart enabled)

---

## 7. Project Structure

```
order-service/
├── src/main/java/com/orderflow/orderservice/
│   ├── controller/     OrderController.java
│   ├── service/        OrderService.java
│   ├── repository/     OrderRepository.java
│   ├── model/          Order.java, OrderItem.java, OrderStatus.java
│   ├── dto/            CreateOrderRequest.java, OrderResponse.java
│   ├── exception/      OrderNotFoundException.java, GlobalExceptionHandler.java
│   └── security/       JwtFilter.java, SecurityConfig.java
├── src/main/resources/
│   └── application.yml
├── src/test/java/
│   ├── controller/     OrderControllerTest.java (MockMvc)
│   ├── service/        OrderServiceTest.java (Mockito)
│   └── integration/    OrderIntegrationTest.java (Testcontainers)
├── cdk/                AWS CDK stack (Lambda + API GW + RDS)
└── build.gradle.kts
```

> **Lambda packaging**: Spring Cloud Function adapter wraps the Spring Boot app as a Lambda handler. Deployed via `./gradlew buildZip` + CDK.

---

## 8. Verification Plan

### Automated Tests
```bash
# Run all tests
./gradlew test

# Run only integration tests (needs Docker running)
./gradlew test --tests "*.integration.*"

# View coverage report
open build/reports/jacoco/test/html/index.html

# Build Lambda ZIP
./gradlew buildZip
```

### Manual Verification
1. Start locally: `./gradlew bootRun`
2. Open Swagger UI: [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)
3. Authenticate → Create an order → Retrieve it → Update status → Cancel it
4. Deploy: `cdk deploy` → test via API Gateway URL

---

> ✅ **Ready to scaffold?** Review and approve this spec, then say "looks good" and I'll generate the full Lambda-based Spring Boot project.
