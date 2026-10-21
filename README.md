# Scaler Capstone — Applied E-Commerce Platform

A 7-microservice e-commerce backend built in Java/Spring Boot, plus a Kong API Gateway
configuration, as a single architecture split across 8 repositories (one per deployable
unit, matching the real service boundaries — not merged into a monorepo).

Full writeup, architecture, schema, and test results: see the submitted project report
(`Applied Software Project Report`).

## Services

| Service | Repository | Responsibility |
|---|---|---|
| User Service | [scaler_backend_project_user_service](https://github.com/vohraharsh/scaler_backend_project_user_service) | Registration, profile, password reset (MySQL) |
| Auth Service | [scaler_backend_project_auth_service](https://github.com/vohraharsh/scaler_backend_project_auth_service) | Login (JWT issuance), logout (Redis-backed revocation) |
| Product Service | [scaler_backend_project_product_service](https://github.com/vohraharsh/scaler_backend_project_product_service) | Catalog, categories, Elasticsearch-backed search (MySQL + Elasticsearch) |
| Cart Service | [scaler_backend_project_cart_service](https://github.com/vohraharsh/scaler_backend_project_cart_service) | Shopping cart (MongoDB, Redis-cached active-cart lookup) |
| Order Service | [scaler_backend_project_order_service](https://github.com/vohraharsh/scaler_backend_project_order_service) | Checkout orchestration, order history, delivery tracking (MySQL) |
| Payment Service | [scaler_backend_project_payment_service](https://github.com/vohraharsh/scaler_backend_project_payment_service) | Mock payment gateway, payment records (MySQL) |
| Notification Service | [scaler_backend_project_notification_service](https://github.com/vohraharsh/scaler_backend_project_notification_service) | Kafka consumer — registration/cart/order notifications (mocked) |
| Kong Gateway Config | [scaler_backend_project_kong_file](https://github.com/vohraharsh/scaler_backend_project_kong_file) | Declarative `kong.yml` — routing + JWT verification at the edge |

## Architecture at a Glance

- **Communication:** synchronous REST for request-path calls (checkout orchestration),
  asynchronous Kafka events for side effects (notifications).
- **Auth:** JWT (HS256), verified independently by Kong and by each protected service,
  with a Redis-backed blocklist for logout/token revocation.
- **Datastores:** MySQL (User, Product, Order, Payment), MongoDB (Cart), Redis (cache +
  revocation), Elasticsearch (product search).
- **Testing:** 146 JUnit 5 + Mockito unit test cases across all 7 services, all passing,
  plus manual end-to-end verification against the live services for cross-service
  behaviour unit tests can't cover.

## How to Run Locally

Each service is independent — clone the ones you need, each with its own
`application.properties` (local MySQL/MongoDB/Redis/Kafka/Elasticsearch required) and
run as a standard Spring Boot application. Start Kong last, pointed at `kong.yml` in
the gateway-config repo, so it can route to the already-running services.
