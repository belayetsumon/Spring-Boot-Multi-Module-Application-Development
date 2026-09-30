# Spring Boot Multi-Module Learning Project — Full Copy-Paste Guide

This guide is intentionally written so an intermediate-level Java developer can create the project from an empty folder by copying each file exactly into the shown path.

## Technology Stack

- Java 17+
- Spring Boot 3.x
- Maven multi-module project
- Spring MVC
- Spring Data JPA / Hibernate
- PostgreSQL
- Thymeleaf
- Bootstrap 5
- JavaScript
- Fetch API
- Bean Validation
- Spring transactions
- Spring application events
- SOLID principles
- Modular monolith architecture

## Business Flow

```text
Customer
   ↓
Product
   ↓
Inventory
   ↓
Sales Order
   ↓
OrderPlacedEvent
   ↓
Notification
```

## Core Architecture Rule

Never access another module's repository directly.

Correct:

```text
Sales
  ↓
ProductQueryService
  ↓
ProductService
  ↓
ProductRepository
```

Wrong:

```text
Sales
  ↓
ProductRepository
```

## Folder Structure

```text
mini-order-modular/
├── pom.xml
├── common/
├── customer/
├── product/
├── inventory/
├── sales/
├── notification/
└── app/
```

## PostgreSQL Setup

Create the database:

```sql
CREATE DATABASE mini_order;
```

For learning, the application properties below use:

```text
username: postgres
password: postgres
```

Change those values to match your local PostgreSQL installation.

## How to Use This Document

1. Create a folder named `mini-order-modular`.
2. Create every file path shown below.
3. Copy the code block under that path into the file.
4. Create the PostgreSQL database.
5. Run `mvn clean install`.
6. Start the application from the `app` module.
7. Open `http://localhost:8080`.

---

# Complete Source Code


## `pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.5</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>mini-order-modular</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>Mini Order Modular</name>
    <description>Learning project for Spring Boot modular monolith architecture</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <modules>
        <module>common</module>
        <module>customer</module>
        <module>product</module>
        <module>inventory</module>
        <module>sales</module>
        <module>notification</module>
        <module>app</module>
    </modules>
</project>
```


## `common/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>com.example</groupId>
            <artifactId>mini-order-modular</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </parent>
        <artifactId>common</artifactId>
        <dependencies>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

        </dependencies>
    </project>
```


## `customer/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>com.example</groupId>
            <artifactId>mini-order-modular</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </parent>
        <artifactId>customer</artifactId>
        <dependencies>

<dependency>
    <groupId>com.example</groupId>
    <artifactId>common</artifactId>
    <version>${project.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

                <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    </project>
```


## `product/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>com.example</groupId>
            <artifactId>mini-order-modular</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </parent>
        <artifactId>product</artifactId>
        <dependencies>

<dependency>
    <groupId>com.example</groupId>
    <artifactId>common</artifactId>
    <version>${project.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

                <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    </project>
```


## `inventory/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>com.example</groupId>
            <artifactId>mini-order-modular</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </parent>
        <artifactId>inventory</artifactId>
        <dependencies>

<dependency>
    <groupId>com.example</groupId>
    <artifactId>common</artifactId>
    <version>${project.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

        
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>product</artifactId>
            <version>${project.version}</version>
        </dependency>
            <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    </project>
```


## `sales/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>com.example</groupId>
            <artifactId>mini-order-modular</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </parent>
        <artifactId>sales</artifactId>
        <dependencies>

<dependency>
    <groupId>com.example</groupId>
    <artifactId>common</artifactId>
    <version>${project.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<dependency>
    <groupId>com.example</groupId>
    <artifactId>customer</artifactId>
    <version>${project.version}</version>
</dependency>
<dependency>
    <groupId>com.example</groupId>
    <artifactId>product</artifactId>
    <version>${project.version}</version>
</dependency>
<dependency>
    <groupId>com.example</groupId>
    <artifactId>inventory</artifactId>
    <version>${project.version}</version>
</dependency>

                <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    </project>
```


## `notification/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>com.example</groupId>
            <artifactId>mini-order-modular</artifactId>
            <version>1.0.0-SNAPSHOT</version>
        </parent>
        <artifactId>notification</artifactId>
        <dependencies>

<dependency>
    <groupId>com.example</groupId>
    <artifactId>common</artifactId>
    <version>${project.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<dependency>
    <groupId>com.example</groupId>
    <artifactId>sales</artifactId>
    <version>${project.version}</version>
</dependency>

        </dependencies>
    </project>
```


## `app/pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>mini-order-modular</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>app</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>customer</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>product</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>inventory</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>sales</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>notification</artifactId>
            <version>${project.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```


## `app/src/main/java/com/example/miniorders/MiniOrderApplication.java`

```java
package com.example.miniorders;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MiniOrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(MiniOrderApplication.class, args);
    }
}
```


## `app/src/main/java/com/example/miniorders/web/GlobalExceptionHandler.java`

```java
package com.example.miniorders.web;

import com.example.miniorders.common.ApiError;
import com.example.miniorders.common.BusinessException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.time.Instant;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ApiError> handleBusiness(BusinessException ex) {
        return ResponseEntity.badRequest()
                .body(new ApiError(Instant.now(), 400, ex.getMessage()));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiError> handleOther(Exception ex) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(new ApiError(Instant.now(), 500, "Unexpected server error."));
    }
}
```


## `app/src/main/java/com/example/miniorders/web/HomeController.java`

```java
package com.example.miniorders.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```


## `app/src/main/resources/application.properties`

```properties
spring.application.name=mini-order-modular

spring.datasource.url=jdbc:postgresql://localhost:5432/mini_order
spring.datasource.username=postgres
spring.datasource.password=postgres

spring.jpa.hibernate.ddl-auto=update
spring.jpa.open-in-view=false
spring.jpa.properties.hibernate.format_sql=true

spring.thymeleaf.cache=false

logging.level.org.hibernate.SQL=DEBUG
logging.level.com.example.miniorders=INFO
```


## `app/src/main/resources/templates/customer/index.html`

```html
<!doctype html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Customers</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">

<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
  <div class="container">
    <a class="navbar-brand" href="/">Mini Order Modular</a>
    <div class="navbar-nav">
      <a class="nav-link" href="/customers">Customers</a>
      <a class="nav-link" href="/products">Products</a>
      <a class="nav-link" href="/inventory">Inventory</a>
      <a class="nav-link" href="/sales">Sales</a>
    </div>
  </div>
</nav>

<main class="container">
  <div class="row g-4">
    <div class="col-lg-4">
      <div class="card shadow-sm">
        <div class="card-header">Add Customer</div>
        <div class="card-body">
          <form method="post" th:action="@{/customers}" th:object="${form}">
            <div class="mb-3">
              <label class="form-label">Code</label>
              <input class="form-control" th:field="*{code}">
              <div class="text-danger small" th:errors="*{code}"></div>
            </div>
            <div class="mb-3">
              <label class="form-label">Name</label>
              <input class="form-control" th:field="*{name}">
              <div class="text-danger small" th:errors="*{name}"></div>
            </div>
            <div class="mb-3">
              <label class="form-label">Email</label>
              <input class="form-control" th:field="*{email}">
              <div class="text-danger small" th:errors="*{email}"></div>
            </div>
            <button class="btn btn-primary">Save Customer</button>
          </form>
        </div>
      </div>
    </div>
    <div class="col-lg-8">
      <div class="card shadow-sm">
        <div class="card-header">Customers</div>
        <div class="table-responsive">
          <table class="table table-striped mb-0">
            <thead><tr><th>ID</th><th>Code</th><th>Name</th><th>Email</th></tr></thead>
            <tbody>
              <tr th:each="c : ${customers}">
                <td th:text="${c.id}"></td>
                <td th:text="${c.code}"></td>
                <td th:text="${c.name}"></td>
                <td th:text="${c.email}"></td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</main>
</body>
</html>
```


## `app/src/main/resources/templates/home.html`

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Mini Order Modular</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">

<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
  <div class="container">
    <a class="navbar-brand" href="/">Mini Order Modular</a>
    <div class="navbar-nav">
      <a class="nav-link" href="/customers">Customers</a>
      <a class="nav-link" href="/products">Products</a>
      <a class="nav-link" href="/inventory">Inventory</a>
      <a class="nav-link" href="/sales">Sales</a>
    </div>
  </div>
</nav>

<main class="container">
  <div class="p-5 bg-white border rounded-3 shadow-sm">
    <h1>Spring Boot Multi-Module Learning Project</h1>
    <p class="lead">Customer → Product → Inventory → Sales → Notification</p>
    <p>Use this application to learn module boundaries, public APIs, transactions, events, Thymeleaf and Fetch.</p>
  </div>
</main>
</body>
</html>
```


## `app/src/main/resources/templates/inventory/index.html`

```html
<!doctype html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Inventory</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">

<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
  <div class="container">
    <a class="navbar-brand" href="/">Mini Order Modular</a>
    <div class="navbar-nav">
      <a class="nav-link" href="/customers">Customers</a>
      <a class="nav-link" href="/products">Products</a>
      <a class="nav-link" href="/inventory">Inventory</a>
      <a class="nav-link" href="/sales">Sales</a>
    </div>
  </div>
</nav>

<main class="container">
  <div class="card shadow-sm mb-4">
    <div class="card-header">Add Stock</div>
    <div class="card-body">
      <form class="row g-3" method="post" action="/inventory/add">
        <div class="col-md-6">
          <label class="form-label">Product</label>
          <select class="form-select" name="productId" required>
            <option value="">Select...</option>
            <option th:each="p : ${products}" th:value="${p.id}" th:text="${p.sku + ' - ' + p.name}"></option>
          </select>
        </div>
        <div class="col-md-3">
          <label class="form-label">Quantity</label>
          <input class="form-control" type="number" name="quantity" min="1" required>
        </div>
        <div class="col-md-3 d-flex align-items-end">
          <button class="btn btn-primary w-100">Add Stock</button>
        </div>
      </form>
    </div>
  </div>

  <div class="card shadow-sm">
    <div class="card-header">Stock</div>
    <div class="table-responsive">
      <table class="table table-striped mb-0">
        <thead><tr><th>Product ID</th><th>Total</th><th>Reserved</th><th>Available</th></tr></thead>
        <tbody>
          <tr th:each="s : ${stocks}">
            <td th:text="${s.productId}"></td>
            <td th:text="${s.quantity}"></td>
            <td th:text="${s.reservedQuantity}"></td>
            <td th:text="${s.availableQuantity}"></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</main>
</body>
</html>
```


## `app/src/main/resources/templates/product/index.html`

```html
<!doctype html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Products</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">

<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
  <div class="container">
    <a class="navbar-brand" href="/">Mini Order Modular</a>
    <div class="navbar-nav">
      <a class="nav-link" href="/customers">Customers</a>
      <a class="nav-link" href="/products">Products</a>
      <a class="nav-link" href="/inventory">Inventory</a>
      <a class="nav-link" href="/sales">Sales</a>
    </div>
  </div>
</nav>

<main class="container">
  <div class="row g-4">
    <div class="col-lg-4">
      <div class="card shadow-sm">
        <div class="card-header">Add Product</div>
        <div class="card-body">
          <form method="post" th:action="@{/products}" th:object="${form}">
            <div class="mb-3"><label class="form-label">SKU</label><input class="form-control" th:field="*{sku}"></div>
            <div class="mb-3"><label class="form-label">Name</label><input class="form-control" th:field="*{name}"></div>
            <div class="mb-3"><label class="form-label">Price</label><input type="number" step="0.01" class="form-control" th:field="*{price}"></div>
            <button class="btn btn-primary">Save Product</button>
          </form>
        </div>
      </div>
    </div>
    <div class="col-lg-8">
      <div class="card shadow-sm">
        <div class="card-header">Products</div>
        <div class="table-responsive">
          <table class="table table-striped mb-0">
            <thead><tr><th>ID</th><th>SKU</th><th>Name</th><th class="text-end">Price</th></tr></thead>
            <tbody>
              <tr th:each="p : ${products}">
                <td th:text="${p.id}"></td>
                <td th:text="${p.sku}"></td>
                <td th:text="${p.name}"></td>
                <td class="text-end" th:text="${p.price}"></td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</main>
</body>
</html>
```


## `app/src/main/resources/templates/sales/index.html`

```html
<!doctype html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Sales Orders</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">

<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
  <div class="container">
    <a class="navbar-brand" href="/">Mini Order Modular</a>
    <div class="navbar-nav">
      <a class="nav-link" href="/customers">Customers</a>
      <a class="nav-link" href="/products">Products</a>
      <a class="nav-link" href="/inventory">Inventory</a>
      <a class="nav-link" href="/sales">Sales</a>
    </div>
  </div>
</nav>

<main class="container">
  <div id="alertBox"></div>

  <div class="card shadow-sm mb-4">
    <div class="card-header">Place Order using JavaScript Fetch</div>
    <div class="card-body">
      <form id="orderForm" class="row g-3">
        <div class="col-md-5">
          <label class="form-label">Customer</label>
          <select id="customerId" class="form-select" required>
            <option value="">Select...</option>
            <option th:each="c : ${customers}" th:value="${c.id}" th:text="${c.code + ' - ' + c.name}"></option>
          </select>
        </div>
        <div class="col-md-4">
          <label class="form-label">Product</label>
          <select id="productId" class="form-select" required>
            <option value="">Select...</option>
            <option th:each="p : ${products}" th:value="${p.id}" th:text="${p.sku + ' - ' + p.name}"></option>
          </select>
        </div>
        <div class="col-md-2">
          <label class="form-label">Qty</label>
          <input id="quantity" type="number" min="1" value="1" class="form-control" required>
        </div>
        <div class="col-md-1 d-flex align-items-end">
          <button class="btn btn-success w-100">Save</button>
        </div>
      </form>
    </div>
  </div>

  <div class="card shadow-sm">
    <div class="card-header">Orders</div>
    <div class="table-responsive">
      <table class="table table-striped mb-0">
        <thead><tr><th>No</th><th>Customer ID</th><th>Total</th><th>Status</th><th>Created</th></tr></thead>
        <tbody>
          <tr th:each="o : ${orders}">
            <td th:text="${o.orderNo}"></td>
            <td th:text="${o.customerId}"></td>
            <td th:text="${o.totalAmount}"></td>
            <td><span class="badge text-bg-success" th:text="${o.status}"></span></td>
            <td th:text="${o.createdAt}"></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</main>

<script>
const form = document.getElementById('orderForm');
const alertBox = document.getElementById('alertBox');

form.addEventListener('submit', async (event) => {
  event.preventDefault();

  const payload = {
    customerId: Number(document.getElementById('customerId').value),
    items: [{
      productId: Number(document.getElementById('productId').value),
      quantity: Number(document.getElementById('quantity').value)
    }]
  };

  try {
    const response = await fetch('/sales/api/orders', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    const data = await response.json();

    if (!response.ok) {
      throw new Error(data.message || 'Order failed');
    }

    alertBox.innerHTML = `<div class="alert alert-success">Order ${data.orderNo} created successfully.</div>`;
    setTimeout(() => window.location.reload(), 600);
  } catch (error) {
    alertBox.innerHTML = `<div class="alert alert-danger">${error.message}</div>`;
  }
});
</script>
</body>
</html>
```


## `common/src/main/java/com/example/miniorders/common/ApiError.java`

```java
package com.example.miniorders.common;

import java.time.Instant;

public record ApiError(
        Instant timestamp,
        int status,
        String message
) {}
```


## `common/src/main/java/com/example/miniorders/common/BusinessException.java`

```java
package com.example.miniorders.common;

public class BusinessException extends RuntimeException {
    public BusinessException(String message) {
        super(message);
    }
}
```


## `customer/src/main/java/com/example/miniorders/customer/api/CustomerQueryService.java`

```java
package com.example.miniorders.customer.api;

import java.util.List;

public interface CustomerQueryService {
    CustomerSummary getRequired(Long id);
    List<CustomerSummary> findAllActive();
}
```


## `customer/src/main/java/com/example/miniorders/customer/api/CustomerSummary.java`

```java
package com.example.miniorders.customer.api;

public record CustomerSummary(
        Long id,
        String code,
        String name,
        String email,
        boolean active
) {}
```


## `customer/src/main/java/com/example/miniorders/customer/domain/Customer.java`

```java
package com.example.miniorders.customer.domain;

import jakarta.persistence.*;

@Entity
@Table(name = "customers", uniqueConstraints = {
        @UniqueConstraint(name = "uk_customer_code", columnNames = "code"),
        @UniqueConstraint(name = "uk_customer_email", columnNames = "email")
})
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 30)
    private String code;

    @Column(nullable = false, length = 120)
    private String name;

    @Column(nullable = false, length = 150)
    private String email;

    @Column(nullable = false)
    private boolean active = true;

    protected Customer() {}

    public Customer(String code, String name, String email) {
        this.code = code;
        this.name = name;
        this.email = email;
    }

    public Long getId() { return id; }
    public String getCode() { return code; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public boolean isActive() { return active; }
}
```


## `customer/src/main/java/com/example/miniorders/customer/internal/CustomerCommandFacade.java`

```java
package com.example.miniorders.customer.internal;

import com.example.miniorders.customer.api.CustomerSummary;
import org.springframework.stereotype.Component;

@Component
public class CustomerCommandFacade {

    private final CustomerService service;

    public CustomerCommandFacade(CustomerService service) {
        this.service = service;
    }

    public CustomerSummary create(String code, String name, String email) {
        return service.create(code, name, email);
    }
}
```


## `customer/src/main/java/com/example/miniorders/customer/internal/CustomerRepository.java`

```java
package com.example.miniorders.customer.internal;

import com.example.miniorders.customer.domain.Customer;
import org.springframework.data.jpa.repository.JpaRepository;

interface CustomerRepository extends JpaRepository<Customer, Long> {
    boolean existsByCodeIgnoreCase(String code);
}
```


## `customer/src/main/java/com/example/miniorders/customer/internal/CustomerService.java`

```java
package com.example.miniorders.customer.internal;

import com.example.miniorders.common.BusinessException;
import com.example.miniorders.customer.api.CustomerQueryService;
import com.example.miniorders.customer.api.CustomerSummary;
import com.example.miniorders.customer.domain.Customer;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional(readOnly = true)
class CustomerService implements CustomerQueryService {

    private final CustomerRepository repository;

    CustomerService(CustomerRepository repository) {
        this.repository = repository;
    }

    @Override
    public CustomerSummary getRequired(Long id) {
        Customer c = repository.findById(id)
                .orElseThrow(() -> new BusinessException("Customer not found: " + id));
        return toSummary(c);
    }

    @Override
    public List<CustomerSummary> findAllActive() {
        return repository.findAll().stream()
                .filter(Customer::isActive)
                .map(this::toSummary)
                .toList();
    }

    @Transactional
    CustomerSummary create(String code, String name, String email) {
        if (repository.existsByCodeIgnoreCase(code)) {
            throw new BusinessException("Customer code already exists.");
        }
        return toSummary(repository.save(new Customer(code.trim(), name.trim(), email.trim())));
    }

    private CustomerSummary toSummary(Customer c) {
        return new CustomerSummary(c.getId(), c.getCode(), c.getName(), c.getEmail(), c.isActive());
    }
}
```


## `customer/src/main/java/com/example/miniorders/customer/web/CustomerController.java`

```java
package com.example.miniorders.customer.web;

import com.example.miniorders.customer.api.CustomerQueryService;
import com.example.miniorders.customer.internal.CustomerCommandFacade;
import jakarta.validation.Valid;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/customers")
public class CustomerController {

    private final CustomerQueryService queryService;
    private final CustomerCommandFacade commandFacade;

    public CustomerController(CustomerQueryService queryService,
                              CustomerCommandFacade commandFacade) {
        this.queryService = queryService;
        this.commandFacade = commandFacade;
    }

    @GetMapping
    public String index(Model model) {
        model.addAttribute("customers", queryService.findAllActive());
        model.addAttribute("form", new CustomerForm());
        return "customer/index";
    }

    @PostMapping
    public String create(@Valid @ModelAttribute("form") CustomerForm form,
                         BindingResult bindingResult,
                         Model model) {
        if (bindingResult.hasErrors()) {
            model.addAttribute("customers", queryService.findAllActive());
            return "customer/index";
        }
        commandFacade.create(form.code(), form.name(), form.email());
        return "redirect:/customers";
    }

    public record CustomerForm(
            @NotBlank @Size(max = 30) String code,
            @NotBlank @Size(max = 120) String name,
            @NotBlank @Email @Size(max = 150) String email
    ) {
        public CustomerForm() { this("", "", ""); }
    }
}
```


## `inventory/src/main/java/com/example/miniorders/inventory/api/InventoryService.java`

```java
package com.example.miniorders.inventory.api;

import java.util.List;

public interface InventoryService {
    void addStock(Long productId, int quantity);
    void reserve(Long productId, int quantity);
    int available(Long productId);
    List<StockView> findAll();
}
```


## `inventory/src/main/java/com/example/miniorders/inventory/api/StockView.java`

```java
package com.example.miniorders.inventory.api;

public record StockView(
        Long id,
        Long productId,
        int quantity,
        int reservedQuantity,
        int availableQuantity
) {}
```


## `inventory/src/main/java/com/example/miniorders/inventory/domain/Stock.java`

```java
package com.example.miniorders.inventory.domain;

import jakarta.persistence.*;

@Entity
@Table(name = "stock", uniqueConstraints = {
        @UniqueConstraint(name = "uk_stock_product", columnNames = "product_id")
})
public class Stock {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "product_id", nullable = false)
    private Long productId;

    @Column(nullable = false)
    private int quantity;

    @Column(name = "reserved_quantity", nullable = false)
    private int reservedQuantity;

    @Version
    private long version;

    protected Stock() {}

    public Stock(Long productId) {
        this.productId = productId;
    }

    public void add(int qty) {
        if (qty <= 0) throw new IllegalArgumentException("Quantity must be positive");
        this.quantity += qty;
    }

    public void reserve(int qty) {
        if (qty <= 0) throw new IllegalArgumentException("Quantity must be positive");
        if (available() < qty) throw new IllegalStateException("Insufficient stock");
        this.reservedQuantity += qty;
    }

    public int available() {
        return quantity - reservedQuantity;
    }

    public Long getId() { return id; }
    public Long getProductId() { return productId; }
    public int getQuantity() { return quantity; }
    public int getReservedQuantity() { return reservedQuantity; }
}
```


## `inventory/src/main/java/com/example/miniorders/inventory/internal/InventoryServiceImpl.java`

```java
package com.example.miniorders.inventory.internal;

import com.example.miniorders.common.BusinessException;
import com.example.miniorders.inventory.api.InventoryService;
import com.example.miniorders.inventory.api.StockView;
import com.example.miniorders.inventory.domain.Stock;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional
class InventoryServiceImpl implements InventoryService {

    private final StockRepository repository;

    InventoryServiceImpl(StockRepository repository) {
        this.repository = repository;
    }

    @Override
    public void addStock(Long productId, int quantity) {
        Stock stock = repository.findByProductId(productId).orElseGet(() -> new Stock(productId));
        stock.add(quantity);
        repository.save(stock);
    }

    @Override
    public void reserve(Long productId, int quantity) {
        Stock stock = repository.findByProductId(productId)
                .orElseThrow(() -> new BusinessException("No stock record for product: " + productId));
        try {
            stock.reserve(quantity);
        } catch (IllegalStateException ex) {
            throw new BusinessException("Insufficient stock for product: " + productId);
        }
    }

    @Override
    @Transactional(readOnly = true)
    public int available(Long productId) {
        return repository.findByProductId(productId).map(Stock::available).orElse(0);
    }

    @Override
    @Transactional(readOnly = true)
    public List<StockView> findAll() {
        return repository.findAll().stream()
                .map(s -> new StockView(
                        s.getId(), s.getProductId(), s.getQuantity(),
                        s.getReservedQuantity(), s.available()))
                .toList();
    }
}
```


## `inventory/src/main/java/com/example/miniorders/inventory/internal/StockRepository.java`

```java
package com.example.miniorders.inventory.internal;

import com.example.miniorders.inventory.domain.Stock;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

interface StockRepository extends JpaRepository<Stock, Long> {
    Optional<Stock> findByProductId(Long productId);
}
```


## `inventory/src/main/java/com/example/miniorders/inventory/web/InventoryController.java`

```java
package com.example.miniorders.inventory.web;

import com.example.miniorders.inventory.api.InventoryService;
import com.example.miniorders.product.api.ProductQueryService;
import jakarta.validation.Valid;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotNull;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping("/inventory")
public class InventoryController {

    private final InventoryService inventoryService;
    private final ProductQueryService productQueryService;

    public InventoryController(InventoryService inventoryService,
                               ProductQueryService productQueryService) {
        this.inventoryService = inventoryService;
        this.productQueryService = productQueryService;
    }

    @GetMapping
    public String index(Model model) {
        model.addAttribute("stocks", inventoryService.findAll());
        model.addAttribute("products", productQueryService.findAllActive());
        return "inventory/index";
    }

    @PostMapping("/add")
    public String add(@Valid @ModelAttribute StockForm form) {
        inventoryService.addStock(form.productId(), form.quantity());
        return "redirect:/inventory";
    }

    public record StockForm(
            @NotNull Long productId,
            @Min(1) int quantity
    ) {}
}
```


## `notification/src/main/java/com/example/miniorders/notification/OrderNotificationListener.java`

```java
package com.example.miniorders.notification;

import com.example.miniorders.sales.api.OrderPlacedEvent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.transaction.event.TransactionPhase;
import org.springframework.transaction.event.TransactionalEventListener;

@Component
public class OrderNotificationListener {

    private static final Logger log = LoggerFactory.getLogger(OrderNotificationListener.class);

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onOrderPlaced(OrderPlacedEvent event) {
        log.info("NOTIFICATION: order {} confirmed for customer {}, total={}",
                event.orderNo(), event.customerId(), event.totalAmount());
    }
}
```


## `product/src/main/java/com/example/miniorders/product/api/ProductQueryService.java`

```java
package com.example.miniorders.product.api;

import java.util.List;

public interface ProductQueryService {
    ProductSummary getRequired(Long id);
    List<ProductSummary> findAllActive();
}
```


## `product/src/main/java/com/example/miniorders/product/api/ProductSummary.java`

```java
package com.example.miniorders.product.api;

import java.math.BigDecimal;

public record ProductSummary(
        Long id,
        String sku,
        String name,
        BigDecimal price,
        boolean active
) {}
```


## `product/src/main/java/com/example/miniorders/product/domain/Product.java`

```java
package com.example.miniorders.product.domain;

import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity
@Table(name = "products", uniqueConstraints = {
        @UniqueConstraint(name = "uk_product_sku", columnNames = "sku")
})
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 40)
    private String sku;

    @Column(nullable = false, length = 150)
    private String name;

    @Column(nullable = false, precision = 19, scale = 2)
    private BigDecimal price;

    @Column(nullable = false)
    private boolean active = true;

    protected Product() {}

    public Product(String sku, String name, BigDecimal price) {
        this.sku = sku;
        this.name = name;
        this.price = price;
    }

    public Long getId() { return id; }
    public String getSku() { return sku; }
    public String getName() { return name; }
    public BigDecimal getPrice() { return price; }
    public boolean isActive() { return active; }
}
```


## `product/src/main/java/com/example/miniorders/product/internal/ProductCommandFacade.java`

```java
package com.example.miniorders.product.internal;

import com.example.miniorders.product.api.ProductSummary;
import org.springframework.stereotype.Component;
import java.math.BigDecimal;

@Component
public class ProductCommandFacade {

    private final ProductService service;

    public ProductCommandFacade(ProductService service) {
        this.service = service;
    }

    public ProductSummary create(String sku, String name, BigDecimal price) {
        return service.create(sku, name, price);
    }
}
```


## `product/src/main/java/com/example/miniorders/product/internal/ProductRepository.java`

```java
package com.example.miniorders.product.internal;

import com.example.miniorders.product.domain.Product;
import org.springframework.data.jpa.repository.JpaRepository;

interface ProductRepository extends JpaRepository<Product, Long> {
    boolean existsBySkuIgnoreCase(String sku);
}
```


## `product/src/main/java/com/example/miniorders/product/internal/ProductService.java`

```java
package com.example.miniorders.product.internal;

import com.example.miniorders.common.BusinessException;
import com.example.miniorders.product.api.ProductQueryService;
import com.example.miniorders.product.api.ProductSummary;
import com.example.miniorders.product.domain.Product;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.util.List;

@Service
@Transactional(readOnly = true)
class ProductService implements ProductQueryService {

    private final ProductRepository repository;

    ProductService(ProductRepository repository) {
        this.repository = repository;
    }

    @Override
    public ProductSummary getRequired(Long id) {
        return repository.findById(id)
                .map(this::toSummary)
                .orElseThrow(() -> new BusinessException("Product not found: " + id));
    }

    @Override
    public List<ProductSummary> findAllActive() {
        return repository.findAll().stream()
                .filter(Product::isActive)
                .map(this::toSummary)
                .toList();
    }

    @Transactional
    ProductSummary create(String sku, String name, BigDecimal price) {
        if (repository.existsBySkuIgnoreCase(sku)) {
            throw new BusinessException("SKU already exists.");
        }
        if (price.signum() < 0) {
            throw new BusinessException("Price cannot be negative.");
        }
        return toSummary(repository.save(new Product(sku.trim(), name.trim(), price)));
    }

    private ProductSummary toSummary(Product p) {
        return new ProductSummary(p.getId(), p.getSku(), p.getName(), p.getPrice(), p.isActive());
    }
}
```


## `product/src/main/java/com/example/miniorders/product/web/ProductController.java`

```java
package com.example.miniorders.product.web;

import com.example.miniorders.product.api.ProductQueryService;
import com.example.miniorders.product.internal.ProductCommandFacade;
import jakarta.validation.Valid;
import jakarta.validation.constraints.DecimalMin;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;

import java.math.BigDecimal;

@Controller
@RequestMapping("/products")
public class ProductController {

    private final ProductQueryService queryService;
    private final ProductCommandFacade commandFacade;

    public ProductController(ProductQueryService queryService,
                             ProductCommandFacade commandFacade) {
        this.queryService = queryService;
        this.commandFacade = commandFacade;
    }

    @GetMapping
    public String index(Model model) {
        model.addAttribute("products", queryService.findAllActive());
        model.addAttribute("form", new ProductForm());
        return "product/index";
    }

    @PostMapping
    public String create(@Valid @ModelAttribute("form") ProductForm form,
                         BindingResult bindingResult,
                         Model model) {
        if (bindingResult.hasErrors()) {
            model.addAttribute("products", queryService.findAllActive());
            return "product/index";
        }
        commandFacade.create(form.sku(), form.name(), form.price());
        return "redirect:/products";
    }

    public record ProductForm(
            @NotBlank @Size(max = 40) String sku,
            @NotBlank @Size(max = 150) String name,
            @NotNull @DecimalMin("0.00") BigDecimal price
    ) {
        public ProductForm() { this("", "", BigDecimal.ZERO); }
    }
}
```


## `sales/src/main/java/com/example/miniorders/sales/api/OrderPlacedEvent.java`

```java
package com.example.miniorders.sales.api;

import java.math.BigDecimal;

public record OrderPlacedEvent(
        Long orderId,
        String orderNo,
        Long customerId,
        BigDecimal totalAmount
) {}
```


## `sales/src/main/java/com/example/miniorders/sales/api/PlaceOrderCommand.java`

```java
package com.example.miniorders.sales.api;

import java.util.List;

public record PlaceOrderCommand(
        Long customerId,
        List<Item> items
) {
    public record Item(Long productId, int quantity) {}
}
```


## `sales/src/main/java/com/example/miniorders/sales/api/SalesOrderService.java`

```java
package com.example.miniorders.sales.api;

import java.util.List;

public interface SalesOrderService {
    SalesOrderView placeOrder(PlaceOrderCommand command);
    List<SalesOrderView> findAll();
}
```


## `sales/src/main/java/com/example/miniorders/sales/api/SalesOrderView.java`

```java
package com.example.miniorders.sales.api;

import java.math.BigDecimal;
import java.time.Instant;

public record SalesOrderView(
        Long id,
        String orderNo,
        Long customerId,
        BigDecimal totalAmount,
        String status,
        Instant createdAt
) {}
```


## `sales/src/main/java/com/example/miniorders/sales/domain/SalesOrder.java`

```java
package com.example.miniorders.sales.domain;

import jakarta.persistence.*;

import java.math.BigDecimal;
import java.time.Instant;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "sales_orders", uniqueConstraints = {
        @UniqueConstraint(name = "uk_sales_order_no", columnNames = "order_no")
})
public class SalesOrder {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "order_no", nullable = false, length = 40)
    private String orderNo;

    @Column(name = "customer_id", nullable = false)
    private Long customerId;

    @Column(name = "total_amount", nullable = false, precision = 19, scale = 2)
    private BigDecimal totalAmount = BigDecimal.ZERO;

    @Column(nullable = false, length = 30)
    private String status;

    @Column(name = "created_at", nullable = false)
    private Instant createdAt;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<SalesOrderItem> items = new ArrayList<>();

    protected SalesOrder() {}

    public SalesOrder(String orderNo, Long customerId) {
        this.orderNo = orderNo;
        this.customerId = customerId;
        this.status = "CONFIRMED";
        this.createdAt = Instant.now();
    }

    public void addItem(Long productId, int quantity, BigDecimal unitPrice) {
        SalesOrderItem item = new SalesOrderItem(this, productId, quantity, unitPrice);
        items.add(item);
        totalAmount = totalAmount.add(item.getLineTotal());
    }

    public Long getId() { return id; }
    public String getOrderNo() { return orderNo; }
    public Long getCustomerId() { return customerId; }
    public BigDecimal getTotalAmount() { return totalAmount; }
    public String getStatus() { return status; }
    public Instant getCreatedAt() { return createdAt; }
}
```


## `sales/src/main/java/com/example/miniorders/sales/domain/SalesOrderItem.java`

```java
package com.example.miniorders.sales.domain;

import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity
@Table(name = "sales_order_items")
public class SalesOrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "order_id", nullable = false)
    private SalesOrder order;

    @Column(name = "product_id", nullable = false)
    private Long productId;

    @Column(nullable = false)
    private int quantity;

    @Column(name = "unit_price", nullable = false, precision = 19, scale = 2)
    private BigDecimal unitPrice;

    @Column(name = "line_total", nullable = false, precision = 19, scale = 2)
    private BigDecimal lineTotal;

    protected SalesOrderItem() {}

    SalesOrderItem(SalesOrder order, Long productId, int quantity, BigDecimal unitPrice) {
        this.order = order;
        this.productId = productId;
        this.quantity = quantity;
        this.unitPrice = unitPrice;
        this.lineTotal = unitPrice.multiply(BigDecimal.valueOf(quantity));
    }

    public BigDecimal getLineTotal() { return lineTotal; }
}
```


## `sales/src/main/java/com/example/miniorders/sales/internal/SalesOrderRepository.java`

```java
package com.example.miniorders.sales.internal;

import com.example.miniorders.sales.domain.SalesOrder;
import org.springframework.data.jpa.repository.JpaRepository;

interface SalesOrderRepository extends JpaRepository<SalesOrder, Long> {
}
```


## `sales/src/main/java/com/example/miniorders/sales/internal/SalesOrderServiceImpl.java`

```java
package com.example.miniorders.sales.internal;

import com.example.miniorders.common.BusinessException;
import com.example.miniorders.customer.api.CustomerQueryService;
import com.example.miniorders.inventory.api.InventoryService;
import com.example.miniorders.product.api.ProductQueryService;
import com.example.miniorders.product.api.ProductSummary;
import com.example.miniorders.sales.api.*;
import com.example.miniorders.sales.domain.SalesOrder;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

@Service
@Transactional
class SalesOrderServiceImpl implements SalesOrderService {

    private static final AtomicLong SEQUENCE = new AtomicLong(1000);

    private final SalesOrderRepository repository;
    private final CustomerQueryService customerQueryService;
    private final ProductQueryService productQueryService;
    private final InventoryService inventoryService;
    private final ApplicationEventPublisher eventPublisher;

    SalesOrderServiceImpl(SalesOrderRepository repository,
                          CustomerQueryService customerQueryService,
                          ProductQueryService productQueryService,
                          InventoryService inventoryService,
                          ApplicationEventPublisher eventPublisher) {
        this.repository = repository;
        this.customerQueryService = customerQueryService;
        this.productQueryService = productQueryService;
        this.inventoryService = inventoryService;
        this.eventPublisher = eventPublisher;
    }

    @Override
    public SalesOrderView placeOrder(PlaceOrderCommand command) {
        if (command == null || command.customerId() == null) {
            throw new BusinessException("Customer is required.");
        }
        if (command.items() == null || command.items().isEmpty()) {
            throw new BusinessException("At least one order item is required.");
        }

        customerQueryService.getRequired(command.customerId());

        SalesOrder order = new SalesOrder(
                "SO-" + SEQUENCE.incrementAndGet(),
                command.customerId()
        );

        for (PlaceOrderCommand.Item line : command.items()) {
            if (line.quantity() <= 0) {
                throw new BusinessException("Quantity must be greater than zero.");
            }

            ProductSummary product = productQueryService.getRequired(line.productId());

            inventoryService.reserve(product.id(), line.quantity());
            order.addItem(product.id(), line.quantity(), product.price());
        }

        SalesOrder saved = repository.save(order);

        eventPublisher.publishEvent(new OrderPlacedEvent(
                saved.getId(),
                saved.getOrderNo(),
                saved.getCustomerId(),
                saved.getTotalAmount()
        ));

        return toView(saved);
    }

    @Override
    @Transactional(readOnly = true)
    public List<SalesOrderView> findAll() {
        return repository.findAll().stream()
                .map(this::toView)
                .toList();
    }

    private SalesOrderView toView(SalesOrder order) {
        return new SalesOrderView(
                order.getId(),
                order.getOrderNo(),
                order.getCustomerId(),
                order.getTotalAmount(),
                order.getStatus(),
                order.getCreatedAt()
        );
    }
}
```


## `sales/src/main/java/com/example/miniorders/sales/web/SalesOrderController.java`

```java
package com.example.miniorders.sales.web;

import com.example.miniorders.customer.api.CustomerQueryService;
import com.example.miniorders.product.api.ProductQueryService;
import com.example.miniorders.sales.api.PlaceOrderCommand;
import com.example.miniorders.sales.api.SalesOrderService;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;

@Controller
@RequestMapping("/sales")
public class SalesOrderController {

    private final SalesOrderService salesOrderService;
    private final CustomerQueryService customerQueryService;
    private final ProductQueryService productQueryService;

    public SalesOrderController(SalesOrderService salesOrderService,
                                CustomerQueryService customerQueryService,
                                ProductQueryService productQueryService) {
        this.salesOrderService = salesOrderService;
        this.customerQueryService = customerQueryService;
        this.productQueryService = productQueryService;
    }

    @GetMapping
    public String index(Model model) {
        model.addAttribute("orders", salesOrderService.findAll());
        model.addAttribute("customers", customerQueryService.findAllActive());
        model.addAttribute("products", productQueryService.findAllActive());
        return "sales/index";
    }

    @PostMapping("/api/orders")
    @ResponseBody
    @ResponseStatus(HttpStatus.CREATED)
    public Object placeOrder(@RequestBody PlaceOrderCommand command) {
        return salesOrderService.placeOrder(command);
    }
}
```


---

# Build and Run

From the project root:

```bash
mvn clean install
```

Run the Spring Boot application:

```bash
mvn -pl app spring-boot:run
```

Alternative:

```bash
cd app
mvn spring-boot:run
```

Open:

```text
http://localhost:8080
```

## Recommended Practice Flow

### 1. Create Customer

Open:

```text
http://localhost:8080/customers
```

Example:

```text
Code: CUST-001
Name: Rahim Trading
Email: rahim@example.com
```

### 2. Create Product

Open:

```text
http://localhost:8080/products
```

Example:

```text
SKU: PRD-001
Name: Business Laptop
Price: 80000
```

### 3. Add Inventory

Open:

```text
http://localhost:8080/inventory
```

Add:

```text
Product: PRD-001
Quantity: 10
```

Expected:

```text
Total: 10
Reserved: 0
Available: 10
```

### 4. Create Sales Order

Open:

```text
http://localhost:8080/sales
```

Select:

```text
Customer: CUST-001
Product: PRD-001
Quantity: 2
```

Expected result:

```text
Order status: CONFIRMED
Total stock: 10
Reserved: 2
Available: 8
```

The notification module should log the committed order event.

---

# Why This Architecture Follows SOLID

## Single Responsibility Principle

Controller:

```text
HTTP request / response
```

Service:

```text
business use case
```

Repository:

```text
persistence
```

Domain:

```text
business state and behavior
```

## Open/Closed Principle

You can add another notification implementation later without changing Sales.

## Liskov Substitution Principle

Any implementation of a module API must preserve the behavior promised by its interface.

## Interface Segregation Principle

Expose small interfaces such as:

```text
CustomerQueryService
ProductQueryService
InventoryService
SalesOrderService
```

rather than one giant application service.

## Dependency Inversion Principle

Sales depends on:

```java
ProductQueryService
```

instead of:

```java
ProductRepository
```

---

# Why Cross-Module Entity References Are Avoided

Avoid:

```java
@ManyToOne
private Product product;
```

inside the Sales module.

Prefer:

```java
private Long productId;
```

Sales then asks Product through its public API.

This keeps ownership clear and avoids tight JPA coupling.

---

# Thymeleaf vs Fetch

Use Thymeleaf for:

- initial page rendering
- standard forms
- server-side validation messages
- tables
- navigation

Use Fetch for:

- async actions
- JSON APIs
- interactions where full-page reload is undesirable

Do not move business rules into JavaScript.

The server remains authoritative.

---

# Transaction Rule

Order placement runs in one Spring transaction:

```text
validate customer
    ↓
load product
    ↓
reserve stock
    ↓
save order
    ↓
publish event
    ↓
commit
```

If stock reservation fails, the order should not be committed.

---

# Event Rule

Sales publishes:

```text
OrderPlacedEvent
```

Notification listens after commit.

That prevents sending a successful order notification before the database transaction is actually committed.

---

# What to Add Next

After this base project works, add these features gradually:

1. Customer edit/deactivate
2. Product edit/deactivate
3. Multi-line sales orders
4. Search and pagination
5. Better Bootstrap validation messages
6. Flyway migrations
7. Spring Security
8. Login and roles
9. CSRF-aware Fetch
10. ArchUnit module boundary tests
11. Testcontainers PostgreSQL tests
12. Audit fields
13. Order cancellation
14. Stock release
15. Idempotency
16. Notification history
17. Spring Modulith
18. Outbox pattern
19. RabbitMQ/Kafka only after the modular monolith is understood

---

# Architecture Rules to Remember

```text
1. Each module owns its business entities.
2. Each module owns its repositories.
3. Repositories are never public integration APIs.
4. Other modules communicate through small public interfaces.
5. Cross-module JPA relationships should normally be avoided.
6. Use direct calls when an immediate result is required.
7. Use events for facts that already happened.
8. Keep the common module small.
9. Put transactions around business use cases, not controllers.
10. Keep JavaScript responsible for UI interaction, not business truth.
11. Keep dependency direction explicit.
12. Avoid circular module dependencies.
```

---

# Final Learning Goal

After completing this project, you should be able to explain and demonstrate:

```text
Maven module
Business module
Modular monolith
Module ownership
Public module API
Repository encapsulation
Cross-module communication
Transaction boundary
Application event
AFTER_COMMIT event
DTO boundary
SOLID principles
Thymeleaf rendering
Fetch API integration
PostgreSQL persistence
Future microservice extraction
```

That understanding is the foundation needed before designing a much larger modular Back Office, ERP, CRM, Inventory, Finance, Procurement, HR, or eCommerce system.
