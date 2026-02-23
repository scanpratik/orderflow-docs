# 📋 SDD Spec — Order Service (OrderFlow v1)

> **Methodology**: Spec-Driven Development — review and approve this before any code is written.  
> **Project**: OrderFlow · **Service**: order-service · **Phase**: Week 2–4  
> **Compute**: AWS Lambda (via Spring Cloud Function)

---

## 1. What Are We Building?

A **production-grade REST API** for managing orders, deployed as an **AWS Lambda function** behind API Gateway.  
It demonstrates: clean layered architecture, DynamoDB persistence, input validation, JWT auth, global error handling, and OpenAPI docs — all running serverless on Lambda.

---

## 2. Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Java 21 (virtual threads enabled) |
| Framework | Spring Boot 3.4.x + **Spring Cloud Function** |
| Build | Gradle (Kotlin DSL) |
| Database | **DynamoDB** (AWS managed, serverless, no VPC needed) |
| ORM | AWS DynamoDB Enhanced Client (SDK v2) |
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

### DynamoDB Table: `orders`

| Attribute | Type | Notes |
|-----------|------|-------|
| `id` (PK) | String (UUID) | Partition key, auto-generated |
| `customerId` | String | Required |
| `status` | String (ENUM) | `PENDING` \| `CONFIRMED` \| `SHIPPED` \| `DELIVERED` \| `CANCELLED` |
| `totalAmount` | Number (Decimal) | Computed from items |
| `items` | List\<Map\> | Embedded — see below |
| `createdAt` | String (ISO-8601) | Auto-set on write |
| `updatedAt` | String (ISO-8601) | Auto-updated on write |

### Embedded `items` List (within each order document)

| Attribute | Type | Notes |
|-----------|------|-------|
| `itemId` | String (UUID) | Auto-generated |
| `productId` | String | Required |
| `quantity` | Number | Min: 1 |
| `unitPrice` | Number | Min: 0.01 |

> **Design note**: Items are embedded in the order document (DynamoDB document model). No separate `order_items` table — this avoids multi-table transactions and aligns with DynamoDB best practices for 1:few relationships.
>
> **GSI**: `customerId-index` (GSI on `customerId`) for querying all orders by customer.

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
├── cdk/                AWS CDK stack (Lambda + API GW + DynamoDB)
└── build.gradle.kts
```

> **Lambda packaging**: Spring Cloud Function adapter wraps the Spring Boot app as a Lambda handler. Deployed via `./gradlew buildZip` + CDK.  
> **DynamoDB**: Provisioned via CDK (`TableV2` with PAY_PER_REQUEST billing). Lambda IAM role gets `dynamodb:PutItem`, `dynamodb:GetItem`, `dynamodb:Query`, `dynamodb:UpdateItem` on the table.

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
```

### Deployment to AWS Lambda
```bash
# Build deployable ZIP
./gradlew buildZip

# Deploy via CDK (creates Lambda + API GW + DynamoDB table)
cdk deploy

# API Gateway URL will be printed — test live endpoint
curl https://<api-id>.execute-api.<region>.amazonaws.com/prod/api/v1/orders
```

### Manual Verification
1. Start locally: `./gradlew bootRun`
2. Open Swagger UI: [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)
3. Authenticate → Create an order → Retrieve it → Update status → Cancel it
4. Deploy: `cdk deploy` → test via API Gateway URL

---

> ✅ **Ready to scaffold?** Review and approve this spec, then say "looks good" and I'll generate the full Lambda-based Spring Boot project.
