# Systems Engineering & Production Backend Master Guide (A to Z)

> **Objective:** Transition from being a *laptop coder who builds features locally* to a *systems engineer who builds, deploys, and maintains reliable, production-grade distributed backends.*

---

## The Mental Model: Localhost vs. Production Reality

```
+-----------------------------------------------------------------------------------------+
|                                    THE TWO WORLDS                                       |
+-----------------------------------------------------------------------------------------+
| LOCALHOST (Laptop Coder):                                                               |
|  - One user at a time (You).                                                            |
|  - Zero network latency; local SQLite / local Redis never drops connection.             |
|  - Manual browser testing; bugs fixed casually after 20 refreshes.                     |
|  - Restarting the server clears state and fixes weird memory/cache bugs.                |
|                                                                                         |
| PRODUCTION (Systems Engineer):                                                          |
|  - Multiple concurrent users hitting the same endpoints at the exact same millisecond.   |
|  - Network calls fail randomly; external APIs (Paystack, SendGrid) drop packets.        |
|  - Webhooks deliver out-of-order or duplicate payloads.                                 |
|  - Databases lock, queues back up, and memory leaks bring down servers.                 |
|  - Bugs cost real money and user trust. Code must be tested, observable, and resilient. |
+-----------------------------------------------------------------------------------------+
```

---

## Table of Contents
1. [Pillar 1: Advanced Database Engineering (Beyond Simple ORM)](#pillar-1-advanced-database-engineering-beyond-simple-orm)
2. [Pillar 2: Asynchronous Systems & Distributed Tasks](#pillar-2-asynchronous-systems--distributed-tasks)
3. [Pillar 3: Payments, Webhooks & Financial Integrity](#pillar-3-payments-webhooks--financial-integrity)
4. [Pillar 4: Infrastructure, DevOps & Containerization](#pillar-4-infrastructure-devops--containerization)
5. [Pillar 5: Automated Testing & Reliability](#pillar-5-automated-testing--reliability)
6. [Pillar 6: Observability, Logging & Security](#pillar-6-observability-logging--security)
7. [Pillar 7: High-Yield System Design & Core DSA](#pillar-7-high-yield-system-design--core-dsa)
8. [Pillar 8: The High-Speed Execution & Survival Blueprint](#pillar-8-the-high-speed-execution--survival-blueprint)
9. [Master 90-Day Interactive Checklist](#master-90-day-interactive-checklist)

---

## Pillar 1: Advanced Database Engineering (Beyond Simple ORM)

Junior developers think in Django models and rely blindly on the ORM. Systems engineers think in PostgreSQL internals, table locks, and disk I/O.

### 1.1 Relational Database Internals & Indexing
* **Sequential Scan vs. Index Scan:** Without an index, the database reads every single row on disk ($O(N)$). With a B-Tree index, lookups take $O(\log N)$ time.
* **B-Tree Indexes:** Default for equality (`=`) and range (`<`, `>`, `BETWEEN`) queries.
* **Composite Indexes (Multi-Column):** Order matters. If you index `(user_id, created_at)`:
  * Fast: `WHERE user_id = 5 AND created_at > ...`
  * Fast: `WHERE user_id = 5`
  * Slow (won't use index): `WHERE created_at > ...`
* **Query Execution Analysis:** Always inspect real SQL plans in PostgreSQL:
  ```sql
  EXPLAIN ANALYZE SELECT * FROM orders_order WHERE user_id = 'uuid-here';
  ```
  Look for:
  * `Seq Scan` on large tables (Danger: missing index).
  * `Index Scan` or `Bitmap Index Scan` (Good).

### 1.2 Eliminating the N+1 Query Disaster
* **The Problem:** Fetching 100 orders and then iterating over each order's customer makes 1 + 100 = 101 database queries. Under traffic, this destroys database performance.
* **The Solution:**
  * `select_related(*fields)`: Uses a SQL `INNER JOIN` or `LEFT OUTER JOIN`. Best for `ForeignKey` and `OneToOneField`.
  * `prefetch_related(*lookups)`: Runs a second query with `WHERE id IN (...)` and merges in Python. Best for `ManyToManyField` and reverse `ForeignKey`.
  ```python
  # BAD: 1 query for orders + N queries for users
  orders = Order.objects.all()
  for o in orders:
      print(o.user.email)

  # GOOD: Exactly 1 query with SQL JOIN
  orders = Order.objects.select_related('user').prefetch_related('items__product').all()
  ```

### 1.3 ACID & Database Transactions
* **Atomicity:** All operations succeed, or none do.
* **Consistency:** The database transitions only between valid states matching constraints.
* **Isolation:** Concurrent transactions do not corrupt each other.
* **Durability:** Committed data is permanently saved to non-volatile disk.
* **Usage in Django:**
  ```python
  from django.db import transaction

  @transaction.atomic
  def checkout(user, cart):
      # Deduct wallet balance
      # Decrement inventory stock
      # Create order record
      # If ANY step crashes, ALL steps roll back automatically
      pass
  ```

### 1.4 Concurrency & Race Conditions (The Double-Spend Problem)
* **The Scenario:** Two users click "Buy" on the last remaining shirt size at the exact same millisecond. Both read `stock = 1`, both decrement to `0`, both get an order.
* **Pessimistic Locking (`select_for_update`):** Locks the row at the SQL level until the transaction finishes.
  ```python
  with transaction.atomic():
      # Locks this row in PostgreSQL: SELECT ... FOR UPDATE
      variant = ProductVariantSize.objects.select_for_update().get(id=variant_id)
      if variant.stock_quantity < quantity:
          raise ValidationError("Out of stock")
      variant.stock_quantity -= quantity
      variant.save()
  ```
* **Optimistic Locking:** Add a `version` field. Only update if `version` matches what was read; if 0 rows updated, retry.

### 1.5 Connection Pooling
* Creating a new TLS/TCP connection to PostgreSQL on every HTTP request consumes 10–20ms of overhead and spikes database CPU.
* **Solution:**
  * Configure `CONN_MAX_AGE` (e.g. 600 seconds) in Django settings to reuse database connections.
  * In high-concurrency environments, place **PgBouncer** in front of PostgreSQL to pool hundreds of application threads into a small pool of database connections.

---

## Pillar 2: Asynchronous Systems & Distributed Tasks

Real systems never execute slow operations (sending emails, image processing, webhook retries) inside the HTTP request/response cycle.

### 2.1 The Producer-Broker-Consumer Architecture
```
[ User Request ] ===> [ Django View ] (Producer)
                            |
                     (pushes task payload)
                            v
                    [ Redis / RabbitMQ ] (Message Broker Queue)
                            ^
                     (pops and executes)
                            |
                    [ Celery Worker ] (Consumer Process)
```

### 2.2 Task Idempotency (Crucial Rule)
* **Rule:** A background task must produce the exact same outcome whether it runs once, twice, or ten times.
* **Why:** If the network drops right after the worker finishes, the broker may believe the task died and re-dispatch it.
* **Implementation:** Always check state before executing:
  ```python
  @shared_task(bind=True, max_retries=3)
  def send_order_confirmation(self, order_id):
      order = Order.objects.get(id=order_id)
      if order.confirmation_email_sent:
          return "Already sent"
      
      try:
          send_mail(...)
          order.confirmation_email_sent = True
          order.save(update_fields=['confirmation_email_sent'])
      except NetworkError as exc:
          # Exponential backoff retry: 2s, 4s, 8s...
          raise self.retry(exc=exc, countdown=2 ** self.request.retries)
  ```

### 2.3 Dead Letter Queues (DLQ)
* When a task fails repeatedly and exhausts all retries, do not let it disappear silently. Route it to a Dead Letter Queue or trigger an alert in Sentry for manual engineer inspection.

---

## Pillar 3: Payments, Webhooks & Financial Integrity

### 3.1 Webhook Verification (HMAC-SHA512)
* Anyone on the internet can send a `POST` request to `/api/payments/webhook/`. You must cryptographically prove it came from Paystack.
```python
import hmac, hashlib

def verify_paystack_webhook(request_body, signature_header, secret_key):
    computed_hash = hmac.new(
        secret_key.encode('utf-8'),
        msg=request_body,
        digestmod=hashlib.sha512
    ).hexdigest()
    return hmac.compare_digest(computed_hash, signature_header)
```

### 3.2 Handling Webhook Duplicates & Out-of-Order Events
* **Idempotency Table:** Record every processed event ID.
  ```python
  class ProcessedWebhookEvent(models.Model):
      event_id = models.CharField(max_length=255, unique=True, db_index=True)
      processed_at = models.DateTimeField(auto_now_add=True)
  ```
  If `ProcessedWebhookEvent.objects.filter(event_id=event_id).exists()`, return `200 OK` immediately without re-crediting the order.

### 3.3 Reconciliation Cron Worker
* Never rely 100% on webhooks. A network partition can cause Paystack to give up before your server receives the webhook.
* **Scheduled Job (Celery Beat):** Every 15 minutes, fetch all orders in `PENDING` state older than 10 minutes. Call `paystack.transaction.verify(reference)`. If paid, update the order.

---

## Pillar 4: Infrastructure, DevOps & Containerization

### 4.1 Production Docker Architecture
Eliminate local Windows/Linux discrepancies by standardizing on Docker.

* **Multi-Container `docker-compose.yml` Architecture:**
  1. `web`: Gunicorn serving Django application.
  2. `postgres`: Managed relational database with volume persistence.
  3. `redis`: In-memory broker and cache.
  4. `celery_worker`: Background task processing.
  5. `celery_beat`: Cron scheduler.

### 4.2 Automated CI/CD Pipeline (GitHub Actions)
Create `.github/workflows/ci.yml`:
```yaml
name: CI Pipeline

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Linting check
        run: ruff check .
      - name: Run Test Suite
        run: pytest
```

### 4.3 Reverse Proxy & Web Servers
* **Why Nginx / Caddy in front of Gunicorn?**
  * Buffers slow client connections (prevents Gunicorn workers from hanging while a slow mobile client uploads data).
  * Handles SSL/TLS termination.
  * Serves static assets efficiently without touching Python threads.

---

## Pillar 5: Automated Testing & Reliability

Stop manual clicking in the browser. A professional backend engineer verifies functionality with repeatable test suites.

### 5.1 The Testing Pyramid
* **Unit Tests:** Fast, isolated tests for utility functions, model methods, and serializers.
* **Integration Tests:** Testing API views end-to-end with DRF `APITestCase` or `pytest-django`.
* **Mocking:** Replace external calls (Paystack, email APIs) with mocked responses:
  ```python
  from unittest.mock import patch

  @patch('payments.services.paystack.initialize_transaction')
  def test_checkout_creates_pending_order(mock_paystack, api_client, user):
      mock_paystack.return_value = {"status": True, "data": {"authorization_url": "https://..."}}
      response = api_client.post('/api/orders/checkout/', {...})
      assert response.status_code == 201
      assert Order.objects.filter(user=user, status='PENDING').exists()
  ```

---

## Pillar 6: Observability, Logging & Security

### 6.1 Sentry (Real-Time Error Telemetry)
* Do not wait for users to report bugs. Install `sentry-sdk`:
  ```python
  import sentry_sdk
  from sentry_sdk.integrations.django import DjangoIntegration
  from sentry_sdk.integrations.celery import CeleryIntegration

  sentry_sdk.init(
      dsn="YOUR_SENTRY_DSN",
      integrations=[DjangoIntegration(), CeleryIntegration()],
      traces_sample_rate=0.2,
      send_default_pii=False,
  )
  ```

### 6.2 Security Essentials (OWASP)
* **SQL Injection:** Avoid raw string formatting in SQL (`f"SELECT * WHERE name = '{name}'"`). Always use parameterized queries or ORM.
* **CORS:** Restrict `CORS_ALLOWED_ORIGINS` strictly to your frontend domain (`https://marvelam.vercel.app`), never `CORS_ALLOW_ALL_ORIGINS = True` in production.
* **JWT Lifecycle:** Short-lived access tokens (15–30 mins) + rotating refresh tokens stored in secure, `HttpOnly` cookies or memory.

---

## Pillar 7: High-Yield System Design & Core DSA

You do not need to solve 500 LeetCode problems to pass backend interviews. You need mastery of the high-frequency patterns.

### 7.1 High-Yield DSA Patterns (Blind 75 Focus)
1. **Hash Maps & Sets:** Instant $O(1)$ lookups, counting frequencies, grouping anagrams.
2. **Two Pointers:** Sorting and scanning arrays inward (e.g. Container With Most Water, 3Sum).
3. **Sliding Window:** Finding optimal subarrays/substrings without nested loops ($O(N)$ vs $O(N^2)$).
4. **Binary Search:** Searching in sorted spaces in $O(\log N)$ time.
5. **Trees & Graphs (BFS / DFS):** Hierarchy, traversal, and shortest-path logic.

### 7.2 System Design Building Blocks
* **Caching (Redis):** Cache-aside pattern (`check cache -> if miss, query DB -> populate cache -> return`).
* **Cache Invalidation:** Setting TTL (Time-To-Live) and actively clearing cache when underlying data updates.
* **Rate Limiting:** Protecting APIs against abuse using Token Bucket or Leaky Bucket algorithms.

---

## Pillar 8: The High-Speed Execution & Survival Blueprint

### 8.1 How to Study and Progress When Power or Data Cuts Out
1. **Pre-load Your Phone with High-Value Technical PDFs:**
   * *Designing Data-Intensive Applications (DDIA)* by Martin Kleppmann (Read chapters 1, 3, 5, 7 on Data Models, Storage, Replication, Transactions).
   * *Architecture Patterns with Python* (Cosmic Python - available free online).
   * Official Django REST Framework documentation (PDF/ePub format).
2. **Offline Architecture Modeling:**
   * Keep a dedicated physical notebook. Design database schemas, write SQL queries by hand, trace edge cases, and draw system architecture diagrams when the laptop is charging or dead.
3. **Eradicate Phone Doom-Scrolling:**
   * The moment the laptop battery dies, your brain seeks cheap dopamine. Put phone into grayscale mode or lock social apps during working hours. Your phone is a secondary technical workstation.

---

## Master 90-Day Interactive Checklist

Use this checklist to track daily progress. Tick each item off as you master the concept and apply it.

### Phase 1: Database & Reliability Hardening (Days 1–30)
- [ ] Understand B-Tree indexes and analyze queries with `EXPLAIN ANALYZE`.
- [ ] Audit Marvelam codebase for $N+1$ queries using `select_related` and `prefetch_related`.
- [ ] Implement `transaction.atomic()` across all cart checkout and payment update flows.
- [ ] Add `select_for_update()` to cart item stock decrement logic to prevent race conditions.
- [ ] Write 15+ automated tests with `pytest-django` covering cart, order, and payments.
- [ ] Mock Paystack API calls inside tests using `unittest.mock`.

### Phase 2: Docker, CI/CD & Asynchronous Architecture (Days 31–60)
- [ ] Write a clean, multi-stage `Dockerfile` for Django.
- [ ] Build a complete `docker-compose.yml` (Django + Postgres + Redis + Celery).
- [ ] Set up GitHub Actions CI to automatically run linting (`ruff`) and testing (`pytest`) on push.
- [ ] Ensure Celery tasks are completely idempotent with proper retry backoff.
- [ ] Implement a Dead Letter Queue strategy or Sentry alert for failed Celery tasks.
- [ ] Build an automated reconciliation Celery Beat task for Paystack transactions.

### Phase 3: System Design, DSA & Career Packaging (Days 61–90)
- [ ] Implement Redis caching on high-read catalog endpoints with cache invalidation on save.
- [ ] Set up Sentry error tracking on Render for both Django and Celery.
- [ ] Solve the core Blind 75 DSA patterns (Hash Maps, Two Pointers, Sliding Window, Binary Search).
- [ ] Write and publish 1 in-depth technical case study (e.g., *"Engineering an Idempotent E-Commerce Payment Pipeline in Django"*).
- [ ] Modernize CV, GitHub profile, and LinkedIn highlighting production backend capabilities.
- [ ] Actively apply to 5+ targeted remote backend roles daily.
