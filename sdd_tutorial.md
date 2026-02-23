# 🎓 SDD Hands-On Tutorial — Learn By Doing

> **Method**: Each exercise is a real prompt you paste to Antigravity.  
> **Project**: OrderFlow — your actual portfolio project.  
> **Rule**: Don't code anything until the spec is approved. Ever.

---

## What is SDD in One Sentence?

> **Write the spec. Review it. Then let AI generate the code from it.**

SDD has 4 questions you must answer before touching code:

```
1. WHAT  → What are we building? (2–3 sentences)
2. HOW   → Exact API contract (endpoints, request/response, errors)
3. DATA  → Exact data model (table, attributes, types)
4. DONE  → Acceptance criteria (binary pass/fail test cases)
```

---

## Exercise 1 — Read a Spec Like a Senior Engineer

**Goal**: Train your eye to spot gaps in a spec before coding starts.

### Your Task
Open [`sdd_spec_order_service.md`](./sdd_spec_order_service.md) and answer these for each section:

| Section | Question to Ask | Red Flag if Missing |
|---------|----------------|---------------------|
| §1 What | Is the scope crystal clear? | "It manages orders" is too vague |
| §3 API | Is every field typed? | "some metadata" = ❌ |
| §4 Data | Are constraints explicit? | "a number" with no min/max = ❌ |
| §5 Rules | Are edge cases listed? | "it validates input" = ❌ |
| §6 Criteria | Is each one a pass/fail test? | "the API is fast" = ❌ |

### Prompt to Use With Antigravity
```
Review my sdd_spec_order_service.md like a senior engineer.
Identify any gaps, ambiguities, or missing edge cases before we start coding.
```

---

## Exercise 2 — Write Your First Spec From Scratch

**Goal**: Write a full SDD spec for a new feature using the template below.

### Scenario
> Add a `GET /api/v1/orders?customerId=X` endpoint that returns all orders for a customer, paginated, sorted by newest first.

### The SDD Template (copy this every time)

```markdown
## 1. What Are We Building?
[2–3 sentences. Problem, user, constraint.]

## 2. API Contract
Method + Path:
Auth: 
Request params/body:
Response (success):
Response (errors):

## 3. Data
[What DynamoDB attributes/indexes does this need?]

## 4. Business Rules
- Rule 1
- Rule 2

## 5. Acceptance Criteria
- [ ] Criterion 1 (binary, testable)
- [ ] Criterion 2
```

### Prompt to Use With Antigravity
```
Help me write an SDD spec for this new feature using our template:
GET /api/v1/orders?customerId=X — returns all orders for a customer,
paginated (default page 20), sorted by createdAt descending.
Use our existing DynamoDB data model (customerId-index GSI).
```

---

## Exercise 3 — Spec → Code (The Core Loop)

**Goal**: Experience the full SDD loop end-to-end.

### Step 1: Write & approve the spec
Use Exercise 2's output. Read it. Ask yourself: *"Would I approve this as a PR?"*

### Step 2: Hand it to AI
```
Here is my approved SDD spec. Generate the code for:
- OrderController.java (endpoint only)
- DynamoDB query logic in OrderRepository.java
Use our existing project structure in sdd_spec_order_service.md §7.
Do not add anything not in the spec.
```

### Step 3: Review the code against the spec
For every line of generated code, ask: **"Is this in the spec?"**

| Code | In Spec? | Action |
|------|----------|--------|
| `@GetMapping` with exact path | ✅ | Keep |
| Sort by `createdAt` DESC | ✅ | Keep |
| Extra field in response | ❌ | Remove or update spec first |
| Default page size = 20 | ✅ | Keep |

### Step 4: Write the test from the acceptance criteria
Each `- [ ]` in §6 of your spec = one test method. Example:

```java
// AC: "GET /orders?customerId=X returns paginated list, default size 20"
@Test
void getOrdersByCustomer_returnsPaginatedList() { ... }

// AC: "Returns 400 if customerId is missing"  
@Test
void getOrdersByCustomer_missingCustomerId_returns400() { ... }
```

---

## Exercise 4 — Spec an Event-Driven Flow (Phase 2 Preview)

**Goal**: Write a spec for an async event flow, not just a REST endpoint.

### Scenario
> When an order is `CONFIRMED`, publish an `OrderConfirmed` event to EventBridge so the inventory service can reserve stock.

### Event SDD Template

```markdown
## 1. What
[What triggers this? What does the downstream service do with it?]

## 2. Event Contract (AsyncAPI format)
Event name: OrderConfirmed
Producer: order-service
Consumer: inventory-service
Payload:
{
  "orderId": "string (UUID)",
  "customerId": "string",
  "items": [{ "productId": "string", "quantity": "int" }],
  "occurredAt": "ISO-8601 timestamp"
}

## 3. Rules
- Published only when status transitions TO CONFIRMED
- Not published for CANCELLED or PENDING orders
- At-least-once delivery (idempotency key = orderId)

## 4. Acceptance Criteria
- [ ] OrderConfirmed event appears on EventBridge when PUT /status → CONFIRMED
- [ ] Event is NOT published when order is CANCELLED
- [ ] inventory-service receives the event and reserves stock
```

### Prompt to Use With Antigravity
```
Help me write an AsyncAPI-style SDD spec for the OrderConfirmed event:
Trigger: PUT /orders/{id}/status → CONFIRMED
Event: OrderConfirmed → EventBridge
Consumer: inventory-service (reserves stock)
Use the template from sdd_tutorial.md Exercise 4.
```

---

## Exercise 5 — The Spec Review Checklist

Run this checklist on **every spec** before saying "looks good":

```
WHAT
  [ ] Problem stated in 2–3 sentences
  [ ] Scope is clear (what's IN and what's OUT)

API CONTRACT
  [ ] Every field has a type
  [ ] Every field has validation rules (min, max, required/optional)
  [ ] All error responses documented (400, 401, 404, 409...)
  [ ] Auth requirement explicit on every endpoint

DATA MODEL
  [ ] Every attribute named and typed
  [ ] Partition key + sort key explicit (DynamoDB)
  [ ] GSIs listed if needed for query patterns

BUSINESS RULES
  [ ] All state transitions documented
  [ ] Edge cases named (empty list, duplicate, concurrent update)
  [ ] Who calculates derived values (e.g. totalAmount)?

ACCEPTANCE CRITERIA
  [ ] Every criterion is binary (pass or fail — no "should be fast")
  [ ] Every criterion maps to exactly one test
  [ ] Happy path covered
  [ ] At least 2 error/edge case paths covered
```

---

## The Golden Rule

> **Never update code without updating the spec first.**  
> The spec is the contract. Code that isn't in the spec is a bug waiting to be found.

---

## Quick Reference — Prompts to Bookmark

| Goal | Prompt |
|------|--------|
| Start a new feature | `"Help me write an SDD spec for: [feature description]"` |
| Review a spec | `"Review this spec for gaps before we code it"` |
| Generate code from spec | `"Generate code from this approved spec. Stick to the spec exactly."` |
| Generate tests from spec | `"Generate JUnit tests for each acceptance criterion in §6"` |
| Update a feature | `"I want to add X. Update the spec first, then we'll update the code."` |

---

*Next: open [`sdd_spec_order_service.md`](./sdd_spec_order_service.md) and run Exercise 1 right now — it takes 5 minutes and trains the most important skill: reading specs critically.*
