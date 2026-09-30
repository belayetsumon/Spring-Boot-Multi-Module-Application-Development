# Spring Boot Multi-Module Advanced Learning Guide

This guide continues from the base **Mini Order Modular** project and takes an intermediate developer toward advanced modular-monolith design.

The goal is not only to add features. The goal is to understand:

- stronger module boundaries
- production database management
- authentication and authorization
- secure Fetch requests
- architecture enforcement
- integration testing
- auditing
- business workflow state changes
- idempotency
- event reliability
- Spring Modulith
- Outbox pattern
- message brokers

---

# 1. Recommended Learning Order

Use this order:

```text
Phase 1 — Better Business CRUD
    1. Customer edit/deactivate
    2. Product edit/deactivate
    3. Multi-line sales orders
    4. Search and pagination
    5. Better Bootstrap validation messages

Phase 2 — Production Database Discipline
    6. Flyway migrations
    12. Audit fields

Phase 3 — Security
    7. Spring Security
    8. Login and roles
    9. CSRF-aware Fetch

Phase 4 — Architecture Enforcement and Testing
    10. ArchUnit module boundary tests
    11. Testcontainers PostgreSQL tests

Phase 5 — Business Workflow Reliability
    13. Order cancellation
    14. Stock release
    15. Idempotency
    16. Notification history

Phase 6 — Advanced Modular Architecture
    17. Spring Modulith
    18. Outbox pattern
    19. RabbitMQ/Kafka
```

Do not jump directly to Kafka.

First learn clean transactions, module boundaries, events, and reliable persistence.

---

# 2. Customer Edit and Deactivate

## Goal

Learn how to update business data without exposing repositories outside the module.

Add:

```text
Customer update
Customer deactivate
Customer activate
```

Do not delete customers that already have orders.

Prefer state:

```text
ACTIVE
INACTIVE
```

instead of hard delete.

## API

Create:

```java
package com.example.miniorders.customer.api;

public interface CustomerCommandService {

    CustomerSummary update(
            Long id,
            String name,
            String email
    );

    void activate(Long id);

    void deactivate(Long id);
}
```

## Domain

Prefer behavior methods:

```java
public void updateProfile(String name, String email) {
    this.name = name;
    this.email = email;
}

public void activate() {
    this.active = true;
}

public void deactivate() {
    this.active = false;
}
```

Avoid controller code like:

```java
customer.setActive(false);
repository.save(customer);
```

The domain should own its state transition.

## Learning Point

You should understand:

```text
Controller
    ↓
Command Service
    ↓
Domain Behavior
    ↓
Repository
```

---

# 3. Product Edit and Deactivate

Follow the same pattern as Customer.

Add:

```text
Product update
Product activate
Product deactivate
```

## Important Business Rule

A deactivated product should:

```text
remain visible in historical sales orders
not be available for new orders
```

This demonstrates why hard deleting master data is often dangerous.

## Query Rule

```java
findAllActive()
```

should only return active products for order entry.

Historical reports may use:

```java
findAll()
```

or dedicated reporting queries.

---

# 4. Multi-Line Sales Orders

The base version creates one item.

Now support:

```text
Sales Order
    ├── Item 1
    ├── Item 2
    ├── Item 3
    └── Item N
```

## Request DTO

```java
public record PlaceOrderCommand(
        Long customerId,
        List<Item> items
) {
    public record Item(
            Long productId,
            int quantity
    ) {}
}
```

## Browser Payload

```javascript
const payload = {
    customerId: 1,
    items: [
        {
            productId: 10,
            quantity: 2
        },
        {
            productId: 12,
            quantity: 1
        }
    ]
};
```

## UI Pattern

Use a table:

```text
Product            Qty       Action
-----------------------------------
Laptop             2         Remove
Mouse              1         Remove
Keyboard            3         Remove

                         Add Item
```

## JavaScript Example

```javascript
let orderItems = [];

function addItem(productId, productName, quantity) {
    orderItems.push({
        productId: Number(productId),
        productName,
        quantity: Number(quantity)
    });

    renderItems();
}

function removeItem(index) {
    orderItems.splice(index, 1);
    renderItems();
}

function renderItems() {
    const tbody = document.getElementById('orderItems');

    tbody.innerHTML = orderItems.map((item, index) => `
        <tr>
            <td>${item.productName}</td>
            <td>${item.quantity}</td>
            <td>
                <button
                    type="button"
                    class="btn btn-sm btn-danger"
                    onclick="removeItem(${index})">
                    Remove
                </button>
            </td>
        </tr>
    `).join('');
}
```

## Learning Point

Understand the difference between:

```text
page state
```

and:

```text
database state
```

The order line list can exist in JavaScript before being persisted.

---

# 5. Search and Pagination

Large tables should not load every record.

Use:

```java
Pageable
Page<T>
```

## Repository

```java
Page<Customer> findByNameContainingIgnoreCase(
        String name,
        Pageable pageable
);
```

## Service

```java
public Page<CustomerSummary> search(
        String query,
        Pageable pageable
) {
    return repository
            .findByNameContainingIgnoreCase(query, pageable)
            .map(this::toSummary);
}
```

## Controller

```java
@GetMapping
public String index(
        @RequestParam(defaultValue = "") String q,
        @RequestParam(defaultValue = "0") int page,
        Model model
) {

    Pageable pageable =
            PageRequest.of(page, 20, Sort.by("name").ascending());

    Page<CustomerSummary> result =
            customerQueryService.search(q, pageable);

    model.addAttribute("page", result);
    model.addAttribute("q", q);

    return "customer/index";
}
```

## Thymeleaf Pagination

```html
<nav>
    <ul class="pagination">

        <li class="page-item"
            th:classappend="${page.first} ? 'disabled'">

            <a class="page-link"
               th:href="@{/customers(
                   q=${q},
                   page=${page.number - 1}
               )}">
                Previous
            </a>
        </li>

        <li class="page-item">
            <span class="page-link"
                  th:text="${page.number + 1}">
            </span>
        </li>

        <li class="page-item"
            th:classappend="${page.last} ? 'disabled'">

            <a class="page-link"
               th:href="@{/customers(
                   q=${q},
                   page=${page.number + 1}
               )}">
                Next
            </a>
        </li>

    </ul>
</nav>
```

## Learning Point

Pagination belongs in the database query.

Do not load everything and paginate in Java.

---

# 6. Better Bootstrap Validation

You need:

```text
client validation
server validation
business validation
database constraints
```

## Bootstrap Pattern

```html
<div class="mb-3">

    <label class="form-label">
        Customer Name
    </label>

    <input
        class="form-control"
        th:classappend="${#fields.hasErrors('name')} ? 'is-invalid'"
        th:field="*{name}">

    <div
        class="invalid-feedback"
        th:errors="*{name}">
    </div>

</div>
```

## Global Error

```html
<div
    th:if="${errorMessage}"
    class="alert alert-danger"
    th:text="${errorMessage}">
</div>
```

## Success Message

```html
<div
    th:if="${successMessage}"
    class="alert alert-success"
    th:text="${successMessage}">
</div>
```

Use `RedirectAttributes`:

```java
redirectAttributes.addFlashAttribute(
        "successMessage",
        "Customer updated successfully."
);
```

---

# 7. Flyway Database Migrations

Stop using:

```properties
spring.jpa.hibernate.ddl-auto=update
```

Use:

```properties
spring.jpa.hibernate.ddl-auto=validate
```

Add dependency:

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>

<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

Create:

```text
app/src/main/resources/db/migration/
```

Example:

```text
V1__create_customer_table.sql
V2__create_product_table.sql
V3__create_inventory_table.sql
V4__create_sales_tables.sql
```

## Example

```sql
CREATE TABLE customers (
    id BIGSERIAL PRIMARY KEY,
    code VARCHAR(30) NOT NULL,
    name VARCHAR(120) NOT NULL,
    email VARCHAR(150) NOT NULL,
    active BOOLEAN NOT NULL DEFAULT TRUE,

    CONSTRAINT uk_customer_code UNIQUE(code),
    CONSTRAINT uk_customer_email UNIQUE(email)
);
```

## Rule

Never edit an already deployed migration.

Wrong:

```text
Change V3 after production has already used it
```

Correct:

```text
V4__add_customer_phone.sql
```

## Learning Point

Your database schema is versioned source code.

---

# 8. Audit Fields

Most business tables should track:

```text
created_at
created_by
updated_at
updated_by
```

## Base Class

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class AuditableEntity {

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private Instant createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private Instant updatedAt;

    @CreatedBy
    @Column(name = "created_by")
    private String createdBy;

    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;
}
```

Enable:

```java
@EnableJpaAuditing
@SpringBootApplication
public class MiniOrderApplication {
}
```

## Auditor Provider

```java
@Bean
AuditorAware<String> auditorProvider() {
    return () -> Optional.of("SYSTEM");
}
```

Later replace `SYSTEM` with the authenticated user.

---

# 9. Spring Security

Add:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Create configuration:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {

        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(
                    "/css/**",
                    "/js/**",
                    "/images/**",
                    "/login"
                ).permitAll()

                .requestMatchers("/admin/**")
                .hasRole("ADMIN")

                .anyRequest()
                .authenticated()
            )

            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/", true)
                .permitAll()
            )

            .logout(logout -> logout
                .logoutSuccessUrl("/login?logout")
            );

        return http.build();
    }
}
```

## Learning Point

Understand:

```text
Authentication
```

means:

```text
Who are you?
```

Authorization means:

```text
What are you allowed to do?
```

---

# 10. Login and Roles

Create:

```text
User
Role
UserRole
```

Typical roles:

```text
ROLE_ADMIN
ROLE_SALES
ROLE_INVENTORY
ROLE_VIEWER
```

## Suggested Access

```text
ADMIN
    everything

SALES
    customer read
    product read
    sales create/read

INVENTORY
    product read
    inventory manage

VIEWER
    read-only pages
```

## Method Security

Enable:

```java
@EnableMethodSecurity
```

Use:

```java
@PreAuthorize("hasRole('SALES')")
public SalesOrderView placeOrder(...) {
}
```

or:

```java
@PreAuthorize("hasAnyRole('ADMIN', 'INVENTORY')")
public void addStock(...) {
}
```

## Learning Point

Do not depend only on hiding buttons.

Always protect server-side operations.

---

# 11. CSRF-Aware Fetch

Spring Security enables CSRF protection.

For normal forms Thymeleaf can include the token.

For Fetch you must send it.

## Add Meta Tags

```html
<meta
    name="_csrf"
    th:content="${_csrf.token}">

<meta
    name="_csrf_header"
    th:content="${_csrf.headerName}">
```

## JavaScript

```javascript
const csrfToken =
    document.querySelector('meta[name="_csrf"]').content;

const csrfHeader =
    document.querySelector('meta[name="_csrf_header"]').content;

const response = await fetch('/sales/api/orders', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        [csrfHeader]: csrfToken
    },
    body: JSON.stringify(payload)
});
```

## Learning Point

Do not disable CSRF just to make Fetch easier.

---

# 12. ArchUnit Module Boundary Tests

Add:

```xml
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5</artifactId>
    <version>1.3.0</version>
    <scope>test</scope>
</dependency>
```

## Rule Example

```java
@AnalyzeClasses(
    packages = "com.example.miniorders"
)
class ArchitectureTest {

    @ArchTest
    static final ArchRule sales_must_not_access_product_internal =
        noClasses()
            .that()
            .resideInAPackage("..sales..")
            .should()
            .dependOnClassesThat()
            .resideInAPackage("..product.internal..");

}
```

## Add More Rules

```text
customer.internal
product.internal
inventory.internal
sales.internal
```

must not be accessed from other modules.

## Web Layer Rule

```java
@ArchTest
static final ArchRule domain_must_not_depend_on_web =
    noClasses()
        .that()
        .resideInAPackage("..domain..")
        .should()
        .dependOnClassesThat()
        .resideInAPackage("..web..");
```

## Learning Point

Architecture should be executable.

Do not depend only on documentation.

---

# 13. Testcontainers PostgreSQL Tests

H2 is not PostgreSQL.

For important integration tests, test with real PostgreSQL.

Add:

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>
```

## Example

```java
@Testcontainers
@SpringBootTest
class SalesOrderIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres =
        new PostgreSQLContainer<>("postgres:16")
            .withDatabaseName("mini_order_test")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void databaseProperties(
            DynamicPropertyRegistry registry) {

        registry.add(
            "spring.datasource.url",
            postgres::getJdbcUrl
        );

        registry.add(
            "spring.datasource.username",
            postgres::getUsername
        );

        registry.add(
            "spring.datasource.password",
            postgres::getPassword
        );
    }
}
```

## Learning Point

Test the same database engine used in production.

---

# 14. Order Cancellation

Add state:

```text
CONFIRMED
CANCELLED
```

Do not simply delete an order.

## Domain

```java
public void cancel() {

    if ("CANCELLED".equals(status)) {
        throw new BusinessException(
            "Order already cancelled."
        );
    }

    status = "CANCELLED";
}
```

## Service Flow

```text
load order
    ↓
validate cancellation
    ↓
release reserved stock
    ↓
mark order CANCELLED
    ↓
publish OrderCancelledEvent
```

## Learning Point

Business workflows usually change state.

They should not destroy history.

---

# 15. Stock Release

Inventory needs:

```java
void release(Long productId, int quantity);
```

## Domain

```java
public void release(int qty) {

    if (qty <= 0) {
        throw new IllegalArgumentException(
            "Quantity must be positive"
        );
    }

    if (reservedQuantity < qty) {
        throw new IllegalStateException(
            "Cannot release more than reserved"
        );
    }

    reservedQuantity -= qty;
}
```

## Learning Point

Reservation is not the same as physical stock deduction.

Later you may support:

```text
ON_HAND
RESERVED
AVAILABLE
ALLOCATED
SHIPPED
RETURNED
```

---

# 16. Idempotency

Users can click twice.

Browsers retry.

Networks fail.

An API should not accidentally create duplicate orders.

## Header

```text
Idempotency-Key
```

Example:

```text
7dbbdc23-fc35-4d5d-8e38-b02fa61bdd11
```

## Table

```sql
CREATE TABLE idempotency_records (
    id BIGSERIAL PRIMARY KEY,
    idempotency_key VARCHAR(100) NOT NULL UNIQUE,
    request_hash VARCHAR(128),
    response_body TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL
);
```

## Request

```javascript
const idempotencyKey = crypto.randomUUID();

await fetch('/sales/api/orders', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Idempotency-Key': idempotencyKey
    },
    body: JSON.stringify(payload)
});
```

## Service Concept

```text
receive request
    ↓
check idempotency key
    ↓
already processed?
    ├── yes → return previous result
    └── no
          ↓
      process request
          ↓
      store result
```

## Learning Point

Idempotency is essential for:

```text
orders
payments
inventory movements
financial transactions
webhooks
```

---

# 17. Notification History

Logging is not enough.

Create:

```text
notification_log
```

Fields:

```text
id
event_type
recipient
channel
subject
message
status
attempt_count
created_at
sent_at
failure_reason
```

## Status

```text
PENDING
SENT
FAILED
```

## Entity Example

```java
@Entity
@Table(name = "notification_log")
public class NotificationLog {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String eventType;

    private String recipient;

    private String channel;

    private String status;

    private Integer attemptCount;

    private Instant createdAt;

    private Instant sentAt;

    @Column(length = 1000)
    private String failureReason;
}
```

## Learning Point

A production notification subsystem needs observability.

---

# 18. Spring Modulith

Once your packages are already modular, add Spring Modulith.

Dependency example:

```xml
<dependency>
    <groupId>org.springframework.modulith</groupId>
    <artifactId>spring-modulith-starter-core</artifactId>
</dependency>
```

## Why Use It?

It helps with:

```text
module discovery
module verification
events
documentation
integration tests
event publication registry
```

## Application Module

A top-level package can become a module:

```text
customer
product
inventory
sales
notification
```

## Verify Modules

```java
class ModulithArchitectureTest {

    ApplicationModules modules =
        ApplicationModules.of(
            MiniOrderApplication.class
        );

    @Test
    void verifiesModuleStructure() {
        modules.verify();
    }
}
```

## Learning Point

Spring Modulith does not magically create good boundaries.

You need good package design first.

---

# 19. Outbox Pattern

Application events work inside one JVM.

But suppose later you send an event to RabbitMQ or Kafka.

Problem:

```text
database transaction commits
but broker publish fails
```

or:

```text
broker publish succeeds
but database transaction rolls back
```

The Outbox pattern solves this.

## Flow

```text
Business Transaction

save sales order
    +
save outbox event
    ↓
same database transaction commits

Later:

Outbox Publisher
    ↓
reads unpublished events
    ↓
publishes to broker
    ↓
marks event as published
```

## Table

```sql
CREATE TABLE outbox_event (
    id UUID PRIMARY KEY,
    aggregate_type VARCHAR(100) NOT NULL,
    aggregate_id VARCHAR(100) NOT NULL,
    event_type VARCHAR(150) NOT NULL,
    payload JSONB NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL,
    published_at TIMESTAMP WITH TIME ZONE
);
```

## Entity

```java
@Entity
@Table(name = "outbox_event")
public class OutboxEvent {

    @Id
    private UUID id;

    private String aggregateType;

    private String aggregateId;

    private String eventType;

    @Column(columnDefinition = "jsonb")
    private String payload;

    private Instant createdAt;

    private Instant publishedAt;
}
```

## Business Transaction

```java
@Transactional
public SalesOrderView placeOrder(...) {

    SalesOrder order =
        repository.save(...);

    outboxRepository.save(
        OutboxEvent.orderPlaced(order)
    );

    return toView(order);
}
```

## Learning Point

The most important idea is:

```text
business row
+
outbox row
```

commit together.

---

# 20. RabbitMQ / Kafka

Only add a message broker when you understand why you need it.

## RabbitMQ

Good for:

```text
work queues
commands
routing
task distribution
traditional messaging
```

## Kafka

Good for:

```text
event streaming
high-volume event history
multiple consumers
analytics pipelines
event-driven architectures
```

Do not choose Kafka because it is popular.

Choose it because the system requirements justify it.

---

# 21. RabbitMQ Example

Add:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

Configuration:

```java
@Configuration
public class RabbitConfiguration {

    @Bean
    Queue orderQueue() {
        return new Queue(
            "order.events",
            true
        );
    }
}
```

Publisher:

```java
rabbitTemplate.convertAndSend(
    "order.events",
    payload
);
```

Consumer:

```java
@RabbitListener(
    queues = "order.events"
)
public void receive(
        OrderPlacedMessage message) {

    // process event
}
```

---

# 22. Kafka Example

Add:

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

Publisher:

```java
kafkaTemplate.send(
    "order-placed",
    orderId.toString(),
    event
);
```

Consumer:

```java
@KafkaListener(
    topics = "order-placed",
    groupId = "notification-service"
)
public void receive(
        OrderPlacedMessage event) {

    // process event
}
```

---

# 23. Important Distributed-System Problem

Once you use a broker, delivery may be:

```text
at least once
```

This means the same event can arrive more than once.

Therefore the consumer must be idempotent.

Example:

```text
OrderPlaced event ID = EVT-1001
```

Consumer stores:

```text
EVT-1001 processed
```

If the event arrives again:

```text
ignore duplicate
```

This connects two important concepts:

```text
Outbox
+
Consumer Idempotency
```

---

# 24. Recommended Advanced Package Structure

After these upgrades, a module can look like:

```text
sales/
└── src/main/java/com/example/miniorders/sales
    ├── api
    │   ├── command
    │   ├── query
    │   ├── dto
    │   └── event
    │
    ├── application
    │   ├── command
    │   ├── query
    │   └── service
    │
    ├── domain
    │   ├── model
    │   ├── event
    │   └── service
    │
    ├── infrastructure
    │   ├── persistence
    │   ├── messaging
    │   └── configuration
    │
    └── web
        ├── mvc
        └── api
```

You do not need this much structure for a tiny module.

Use it when complexity justifies it.

---

# 25. Command and Query Separation

As the module grows, separate:

```text
commands
```

from:

```text
queries
```

Example:

```java
public interface SalesOrderCommandService {

    SalesOrderView placeOrder(
        PlaceOrderCommand command
    );

    void cancelOrder(Long orderId);
}
```

and:

```java
public interface SalesOrderQueryService {

    SalesOrderView getById(Long id);

    Page<SalesOrderView> search(
        SalesOrderFilter filter,
        Pageable pageable
    );
}
```

This is a lightweight form of CQRS.

It does not require separate databases.

---

# 26. Advanced Transaction Rules

Do not annotate every method with:

```java
@Transactional
```

Use transactions at application use-case boundaries.

Good:

```java
@Transactional
public void cancelOrder(...) {
}
```

Read-only:

```java
@Transactional(readOnly = true)
public SalesOrderView getOrder(...) {
}
```

Be careful with:

```text
calling transactional methods from the same class
```

because Spring proxy behavior may not apply as expected.

---

# 27. Concurrency

Inventory is a concurrency-sensitive area.

Two users can order the same item at the same time.

The sample uses:

```java
@Version
```

for optimistic locking.

You should learn:

```text
optimistic locking
pessimistic locking
retry
conflict handling
```

## Optimistic Locking

```java
@Version
private long version;
```

If two transactions modify the same row:

```text
Transaction A succeeds
Transaction B detects stale version
```

Then your application decides whether to retry or show an error.

---

# 28. Better Order Number Generation

The training project used an in-memory sequence.

That is not production-safe.

Use:

```text
database sequence
```

or:

```text
dedicated numbering service
```

Example PostgreSQL:

```sql
CREATE SEQUENCE sales_order_number_seq
START 1000;
```

Then:

```sql
SELECT nextval(
    'sales_order_number_seq'
);
```

Generate:

```text
SO-2026-001001
```

---

# 29. Soft Delete vs Status

Do not automatically add `deleted = true` everywhere.

Ask:

```text
Does this record represent history?
Can it be deactivated?
Must it remain reportable?
```

For Customer:

```text
ACTIVE / INACTIVE
```

is often better.

For temporary configuration:

```text
soft delete
```

may be appropriate.

For audit-sensitive transactions:

```text
never delete
```

may be the correct rule.

---

# 30. Advanced Error Model

Instead of:

```json
{
  "message": "Bad request"
}
```

use structured errors:

```json
{
  "code": "INSUFFICIENT_STOCK",
  "message": "Insufficient stock for product PRD-001",
  "fieldErrors": [],
  "timestamp": "2026-09-30T12:00:00Z"
}
```

Java:

```java
public record ApiError(
        String code,
        String message,
        List<FieldErrorResponse> fieldErrors,
        Instant timestamp
) {}
```

This is easier for JavaScript clients to handle.

---

# 31. Advanced UI Improvements

After the base screens work, improve:

```text
responsive tables
modal editors
confirmation dialogs
toast messages
loading indicators
disabled submit buttons
empty states
pagination
filters
form field errors
keyboard accessibility
focus management
```

Example submit loading state:

```javascript
const button =
    document.getElementById('saveOrderButton');

button.disabled = true;
button.innerText = 'Saving...';

try {
    await saveOrder();
}
finally {
    button.disabled = false;
    button.innerText = 'Save Order';
}
```

---

# 32. Testing Strategy

Use layers.

## Unit Test

Test domain behavior:

```text
Stock.reserve()
Stock.release()
SalesOrder.cancel()
```

## Repository Test

Test JPA queries.

## Integration Test

Test:

```text
Sales
+
Customer API
+
Product API
+
Inventory
+
PostgreSQL
```

## MVC Test

Test:

```text
request
authorization
validation
view
response
```

## Architecture Test

Test module boundaries with ArchUnit.

## End-to-End Test

Test:

```text
browser
login
customer
product
stock
sales order
cancel order
```

---

# 33. Recommended Advanced Exercises

Complete these exercises one by one.

## Exercise 1

Deactivate a customer.

Try creating an order.

Expected:

```text
rejected
```

## Exercise 2

Add 3 different products to one order.

Expected:

```text
one order
three items
correct total
three stock reservations
```

## Exercise 3

Try ordering more stock than available.

Expected:

```text
no order saved
no partial reservation
```

## Exercise 4

Submit the same idempotency key twice.

Expected:

```text
one order only
same result returned
```

## Exercise 5

Cancel an order.

Expected:

```text
status = CANCELLED
reserved stock released
OrderCancelledEvent published
```

## Exercise 6

Break a module boundary.

Example:

```text
sales imports ProductRepository
```

Expected:

```text
ArchUnit test fails
```

## Exercise 7

Stop RabbitMQ temporarily.

Expected with Outbox:

```text
order transaction still persists
outbox event remains pending
publisher can retry later
```

---

# 34. When to Split Into Microservices

Do not split because the project has many modules.

Consider splitting when there is a real operational reason:

```text
different scaling needs
different deployment cadence
different ownership team
security boundary
technology boundary
very high load
regulatory isolation
independent availability requirement
```

A module is not automatically a microservice.

---

# 35. Modular Monolith vs Microservices

## Modular Monolith

```text
one deployment
one process
clear internal modules
often one database
simple operations
fast development
```

## Microservices

```text
multiple deployments
network communication
distributed transactions
broker/API contracts
service discovery
observability
higher operational cost
```

The advanced learning goal is to build a modular monolith that can be split later if needed.

---

# 36. Final Target Architecture

After completing this guide:

```text
                     Browser
                        │
             Thymeleaf + Bootstrap
                        │
                      Fetch
                        │
                Spring MVC / API
                        │
         ┌──────────────┼──────────────┐
         │              │              │
     Customer        Product         Sales
         │              │              │
         │              │              ├──────────┐
         │              │              │          │
         │          Inventory API      │      Outbox
         │              │              │          │
         └──────────────┴──────────────┘          │
                        │                         │
                    PostgreSQL                    │
                                                  ▼
                                         Outbox Publisher
                                                  │
                                            Rabbit/Kafka
                                                  │
                                      Notification Consumer
```

---

# 37. Final Skills Checklist

After completing all 19 advanced stages, you should understand:

```text
Maven multi-module architecture
module ownership
public module APIs
package encapsulation
SOLID
server-rendered Thymeleaf
Bootstrap UI
Fetch API
Spring Security
roles and authorization
CSRF
Flyway
auditing
pagination
validation
transactions
optimistic locking
business state transitions
idempotency
application events
Spring Modulith
ArchUnit
Testcontainers
Outbox pattern
RabbitMQ
Kafka
consumer idempotency
distributed-system failure modes
modular monolith evolution
microservice extraction
```

---

# 38. Recommended Final Learning Rule

Do not learn the advanced features as isolated technologies.

Always ask:

```text
What problem does this solve?
Which module owns it?
What transaction boundary does it belong to?
What happens if it fails?
How will I test it?
How do I prevent duplicate processing?
How do I keep modules independent?
```

If you can answer those questions before adding a technology, you are moving from intermediate-level development toward advanced software architecture.
