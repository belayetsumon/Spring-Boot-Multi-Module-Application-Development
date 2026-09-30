# Mini Order Modular — Complete Learning Guideline

## 1. Purpose

This project teaches an intermediate Java developer how to build a **Spring Boot modular monolith** using:

- Java 17
- Maven multi-module project
- Spring Boot
- Spring MVC
- Spring Data JPA / Hibernate
- Thymeleaf
- PostgreSQL
- Bootstrap 5
- JavaScript Fetch API
- Bean Validation
- Spring transactions
- Spring application events
- SOLID principles
- Module API boundaries

Business flow:

`Customer -> Product -> Inventory -> Sales Order -> Notification`

The goal is not to create a large ERP. The goal is to learn how to keep modules independent and maintainable.

---

## 2. Architecture

Modules:

```text
mini-order-modular
├── common
├── customer
├── product
├── inventory
├── sales
├── notification
└── app
```

Dependency direction:

```text
app
 ├─ customer
 ├─ product
 ├─ inventory
 ├─ sales
 └─ notification

sales
 ├─ customer API
 ├─ product API
 └─ inventory API

inventory
 └─ product API

notification
 └─ sales event API
```

### Critical rule

A module must not access another module's repository.

Correct:

```text
Sales -> ProductQueryService -> ProductService -> ProductRepository
```

Wrong:

```text
Sales -> ProductRepository
```

Repositories are package-private in this project to make accidental external access harder.

---

## 3. Package Pattern

Every business module uses this pattern:

```text
module
└── src/main/java/com/example/miniorders/<module>
    ├── api
    ├── domain
    ├── internal
    └── web
```

### `api`
Public contract exposed to other modules.

Examples:
- `CustomerQueryService`
- `ProductQueryService`
- `InventoryService`
- `SalesOrderService`
- DTOs / events

### `domain`
Business entities and domain behavior.

Examples:
- `Customer`
- `Product`
- `Stock`
- `SalesOrder`

### `internal`
Implementation details.

Examples:
- repositories
- application services
- module-private orchestration

### `web`
MVC controllers belonging to the module.

---

## 4. SOLID Application

### S — Single Responsibility

`ProductRepository` only persists products.

`ProductService` handles product use cases.

`ProductController` handles HTTP/UI concerns.

Do not put SQL, HTML, business rules and HTTP logic in one class.

### O — Open/Closed

Later you can implement different notification channels without rewriting Sales.

Example:

```java
public interface NotificationSender {
    void send(String message);
}
```

Implementations:

```text
EmailNotificationSender
SmsNotificationSender
InAppNotificationSender
```

### L — Liskov Substitution

Implementations of a public module interface must obey the same contract.

If `InventoryService.reserve()` promises to fail when stock is unavailable, every implementation must preserve that behavior.

### I — Interface Segregation

Prefer:

```java
CustomerQueryService
CustomerCommandService
```

instead of a giant:

```java
CustomerEverythingService
```

### D — Dependency Inversion

Sales depends on:

```java
ProductQueryService
```

not:

```java
ProductRepository
```

That makes Sales depend on an abstraction.

---

## 5. Database Design

Create PostgreSQL database:

```sql
CREATE DATABASE mini_order;
```

Configure:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mini_order
spring.datasource.username=postgres
spring.datasource.password=postgres
```

Learning mode uses:

```properties
spring.jpa.hibernate.ddl-auto=update
```

For production use Flyway or Liquibase instead.

Tables created by the sample:

```text
customers
products
stock
sales_orders
sales_order_items
```

Notice that Sales stores:

```text
customer_id
product_id
```

but does not map JPA relationships to Customer or Product entities.

That is intentional.

Cross-module JPA relationships create tight coupling.

---

## 6. Build and Run

Requirements:

```text
Java 17+
Maven 3.9+
PostgreSQL
```

From project root:

```bash
mvn clean install
```

Run:

```bash
mvn -pl app spring-boot:run
```

or:

```bash
cd app
mvn spring-boot:run
```

Open:

```text
http://localhost:8080
```

Recommended usage sequence:

1. Create Customer.
2. Create Product.
3. Add Inventory.
4. Place Sales Order.
5. Check Inventory reservation.
6. Check application log for notification event.

---

## 7. UI Design

Pages:

```text
/            Dashboard
/customers   Customer management
/products    Product management
/inventory   Stock management
/sales       Sales order management
```

Bootstrap handles:

- responsive grids
- cards
- forms
- tables
- alerts
- badges
- navbar

Use Thymeleaf where the page is naturally server-rendered.

Use Fetch where an interaction benefits from structured async JSON.

The Sales Order page demonstrates Fetch.

---

## 8. Fetch Pattern

```javascript
const response = await fetch('/sales/api/orders', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(payload)
});

const data = await response.json();

if (!response.ok) {
    throw new Error(data.message || 'Operation failed');
}
```

Important rules:

1. Check `response.ok`.
2. Return structured JSON errors from Spring.
3. Validate input on the server.
4. Never trust browser validation alone.
5. Do not use Fetch for every operation just because it exists.

---

## 9. Transaction Boundary

Order placement is transactional:

```java
@Transactional
public SalesOrderView placeOrder(...) {
    ...
}
```

Flow:

```text
validate customer
    ↓
load product
    ↓
reserve inventory
    ↓
save order
    ↓
publish event
```

If reservation fails, the transaction rolls back.

This is appropriate because all modules run in the same application and database.

When modules become separate microservices, this transaction can no longer span all services. That is where Saga / Outbox patterns become relevant.

---

## 10. Event Pattern

Sales publishes:

```java
OrderPlacedEvent
```

Notification listens:

```java
@TransactionalEventListener(
    phase = TransactionPhase.AFTER_COMMIT
)
```

Why AFTER_COMMIT?

Because you normally do not want to send a "confirmed" notification for a transaction that later rolls back.

Good event candidates:

```text
OrderPlaced
OrderCancelled
CustomerRegistered
StockLow
PaymentCompleted
```

Do not use events for every method call.

Use direct calls when an immediate result is required.

Use events when one module announces something that has already happened.

---

## 11. Why Product Is Not a JPA Relationship in Sales

Avoid this:

```java
@ManyToOne
private Product product;
```

inside `sales`.

That makes Sales depend directly on Product's persistence entity.

Use:

```java
private Long productId;
```

and query Product through:

```java
ProductQueryService
```

Benefits:

- clearer ownership
- fewer accidental joins
- less coupling
- easier future service extraction
- easier module testing

---

## 12. Common Module Rule

Keep `common` very small.

Good candidates:

```text
BusinessException
shared technical result types
generic auditing primitives
technical constants
```

Bad candidates:

```text
Customer
Product
SalesOrder
Inventory
Finance rules
```

If `common` becomes the place where all business classes live, your modules are no longer really modular.

---

## 13. Validation Strategy

Use multiple layers.

### UI validation

```html
<input required>
```

### DTO validation

```java
@NotBlank
@Email
@Size(max = 150)
```

### Business validation

```java
if (repository.existsByCodeIgnoreCase(code)) {
    throw new BusinessException("Customer code already exists.");
}
```

### Database constraints

```java
@UniqueConstraint(...)
```

Each layer has a different purpose.

---

## 14. Exception Strategy

Expected business errors:

```java
throw new BusinessException("Insufficient stock");
```

Convert to:

```json
{
  "timestamp": "...",
  "status": 400,
  "message": "Insufficient stock"
}
```

Unexpected exceptions should not expose stack traces or SQL messages to the browser.

Log detailed errors on the server.

---

## 15. Recommended Next Improvements

After the base project works, implement these in order.

### Level 1

- Edit customer
- Edit product
- deactivate records
- multiple order lines
- pagination
- search
- flash messages
- better validation feedback

### Level 2

- Flyway migrations
- audit columns
- Spring Security
- roles
- login
- CSRF-aware Fetch calls
- optimistic locking conflict UI
- integration testing

### Level 3

- ArchUnit dependency tests
- Testcontainers PostgreSQL
- idempotency key for order submission
- proper order number sequence
- stock reservation/release
- cancellation
- notification history
- application audit log

### Level 4

- Spring Modulith
- domain events
- event publication registry
- Outbox pattern
- RabbitMQ/Kafka
- separate databases/services

Do not start Level 4 until the modular monolith is clean.

---

## 16. Architecture Tests You Should Add

Recommended rules:

```text
sales must not access product.internal
sales must not access customer.internal
sales must not access inventory.internal
notification may depend on sales.api only
repositories must stay internal to their own module
domain packages must not depend on web packages
```

ArchUnit is a good tool for enforcing these rules.

Example concept:

```java
noClasses()
    .that().resideOutsideOfPackage("..product..")
    .should().dependOnClassesThat()
    .resideInAPackage("..product.internal..");
```

---

## 17. UI Architecture Rule

Keep browser responsibilities simple.

```text
Thymeleaf
    → page rendering / forms / initial data

Bootstrap
    → presentation / responsive layout

JavaScript
    → interaction

Fetch
    → structured asynchronous API calls

Spring Controller
    → HTTP contract

Application Service
    → use case

Domain
    → business rules

Repository
    → persistence
```

Do not put business decisions in JavaScript.

For example, the browser should not determine whether stock is sufficient.

Server-side Inventory owns that rule.

---

## 18. Production Rules

Before using this architecture in a real business system:

- replace `ddl-auto=update` with Flyway/Liquibase
- enable authentication and authorization
- configure CSRF correctly
- use structured logging
- use database migrations
- use secrets/environment variables
- use DTOs for external APIs
- add integration tests
- add audit history
- define module ownership
- add observability
- add concurrency testing
- define transaction and retry policies

---

## 19. Learning Checklist

You should be able to answer these after finishing the project:

- What is a Maven module?
- What is a business module?
- Who owns an entity?
- Why should another module not use my repository?
- What belongs in an API package?
- When should modules communicate synchronously?
- When should they communicate through events?
- What is a transaction boundary?
- Why should Sales store product ID instead of Product entity?
- Why should `common` remain small?
- How does Dependency Inversion improve modularity?
- How would I extract Inventory into a microservice later?
- How do I prevent circular module dependencies?
- Where should validation live?
- When should the UI use Thymeleaf vs Fetch?

If you can explain and demonstrate these points, you understand the core of a modular Spring Boot architecture.

---

## 20. Recommended Feature Flow for Practice

### Exercise A

Create:

```text
Customer CUST-001
Product PRD-001
Stock 10
```

Place order:

```text
PRD-001 × 2
```

Expected:

```text
total stock = 10
reserved = 2
available = 8
order = CONFIRMED
notification event logged
```

### Exercise B

Try:

```text
PRD-001 × 20
```

Expected:

```text
HTTP 400
Insufficient stock
no order saved
reservation unchanged
```

This exercise proves the transaction boundary.

### Exercise C

Try to inject `ProductRepository` into Sales.

You should discover that it is package-private and inaccessible.

That deliberately teaches the module boundary.

---

## 21. Recommended Real-World Evolution

Start:

```text
Modular Monolith
```

Then, only when operationally necessary:

```text
Customer module
Product module
Inventory module
Sales module
```

can gradually become:

```text
Customer Service
Catalog Service
Inventory Service
Order Service
```

The clean module API you learn in this project makes that transition much easier.

Do not start with microservices just because the final system may become large.

A well-structured modular monolith is usually the better learning and initial delivery model.
