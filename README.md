
# 🎓 پروژه درس تحلیل و طراحی نرم‌افزار پیشرفته

**موضوع:** پلتفرم مدیریت هوشمند دانشگاه
**استاد:** دکتر فیضی
**مدت اجرا:** ۸ هفته
**تیم:** ۸ نفر + هوش مصنوعی (ChatGPT)

---

## 🔥 ۱. چشم‌انداز پروژه

این پروژه شبیه‌سازی یک سیستم واقعی Enterprise است که باید:

* مقیاس‌پذیر باشد
* event-driven باشد
* Microservices واقعی داشته باشد
* از RabbitMQ، Saga، Circuit Breaker استفاده کند
* API Gateway نقطه ورود باشد

هدف: ساخت **معماری حرفه‌ای، industry-level**.

---

## 🚨 ۲. الزامات کلیدی 

```mermaid
graph TD
    A[الزامات کلیدی پروژه] --> B[معماری میکروسرویس]
    A --> C[الگوی Saga در خرید]
    A --> D[الگوی Circuit Breaker در آزمون]
    A --> E[ارتباط Event-driven با RabbitMQ]
    A --> F[ورود از طریق API Gateway]
    style A fill:#ff6b6b,stroke:#333,stroke-width:1.5
```

✔ Microservices
✔ Saga Pattern
✔ Circuit Breaker
✔ RabbitMQ
✔ API Gateway

---

# 📦 ۳. نیازمندی‌ها

### ۳.۱ نیازمندی‌های عملکردی (Functional Requirements)

| کد    | سرویس       | نیازمندی                        | توضیح            |
| ----- | ----------- | ------------------------------- | ---------------- |
| FR-01 | Auth        | ثبت‌نام/ورود                    | JWT              |
| FR-02 | Auth        | صدور JWT                        | Login Token      |
| FR-03 | Booking     | مشاهده منابع                    | اتاق‌ها، کلاس‌ها |
| FR-04 | Booking     | رزرو + جلوگیری از Overbooking   | قفل + چک تداخل   |
| FR-05 | Marketplace | تعریف محصول                     | توسط فروشنده     |
| FR-06 | Marketplace | الگوی Saga                      | خرید چندمرحله‌ای |
| FR-07 | Exam        | ساخت آزمون                      | استاد            |
| FR-08 | Exam        | شرکت در آزمون + Circuit Breaker | اعلان شروع       |
| FR-09 | IoT         | داده زنده سنسور                 | دما و کلاس       |
| FR-10 | IoT         | نقشه شاتل                       | موقعیت GPS       |

---

### ۳.۲ نیازمندی‌های غیرعملکردی (Non-Functional Requirements)

| کد       | عنوان            | پیامد معماری            |
| -------- | ---------------- | ----------------------- |
| NFR-S01  | مقیاس‌پذیری افقی | Stateless بودن سرویس‌ها |
| NFR-MT01 | Multi-Tenancy    | Schema-per-tenant       |
| NFR-P01  | Performance      | Cache و Async           |
| NFR-SE01 | Security         | JWT + RBAC              |
| NFR-R01  | Fault Tolerance  | Saga + Circuit Breaker  |

---

# 🧱 ۴. دیاگرام‌های C4

---

# 📘 Level 1 – System Context

> نمای کلی کاربران و سیستم

```mermaid
flowchart TB
    Student([دانشجو])
    Teacher([استاد])
    Seller([فروشنده])
    Admin([مدیر سیستم])

    APIGateway([API Gateway])
    MainSystem([پلتفرم مدیریت هوشمند دانشگاه])
    MessageBroker([RabbitMQ])
    DBs((پایگاه‌داده سرویس‌ها))
    IoTDevice([سنسور IoT<br/>دما/موقعیت])

    Student --> APIGateway
    Teacher --> APIGateway
    Seller --> APIGateway
    Admin --> APIGateway

    APIGateway --> MainSystem
    MainSystem --> MessageBroker
    MainSystem --> DBs
    IoTDevice --> MainSystem
```

---

# 📗 Level 2 – Container Diagram

> سرویس‌ها + دیتابیس مستقل + RabbitMQ

```mermaid
flowchart TB

    Student([دانشجو])
    Teacher([استاد])
    Seller([فروشنده])
    Admin([مدیر سیستم])

    APIGateway[[API Gateway]]

    subgraph Services [سرویس‌های سیستم]
        AuthService[[Auth Service]]
        BookingService[[Booking Service]]
        MarketplaceService[[Marketplace Service]]
        ExamService[[Exam Service]]
        IoTService[[IoT Service]]
        MessageBroker[(RabbitMQ)]
    end

    AuthDB[(Auth DB)]
    BookingDB[(Booking DB)]
    MarketDB[(Marketplace DB)]
    ExamDB[(Exam DB)]
    IoTDB[(IoT DB)]

    Student --> APIGateway
    Teacher --> APIGateway
    Seller --> APIGateway
    Admin --> APIGateway

    APIGateway --> AuthService
    APIGateway --> BookingService
    APIGateway --> MarketplaceService
    APIGateway --> ExamService
    APIGateway --> IoTService

    AuthService --> AuthDB
    BookingService --> BookingDB
    MarketplaceService --> MarketDB
    ExamService --> ExamDB
    IoTService --> IoTDB

    MarketplaceService --> MessageBroker
    BookingService --> MessageBroker
    ExamService --> MessageBroker
    IoTService --> MessageBroker

    MessageBroker --> MarketplaceService
    MessageBroker --> BookingService
    MessageBroker --> ExamService
    MessageBroker --> IoTService
```

---

# 🧩 Level 3 — Auth Service

> احراز هویت، JWT، Role Management

```mermaid
flowchart TB

    subgraph AuthService ["Auth Service"]
        Controller[[Auth Controller]]
        ServiceLayer[[Auth Logic]]
        UserRepo[(User Repo)]
        RoleRepo[(Role Repo)]
        JWTProvider[[JWT Provider]]
        PasswordHasher[[Password Hasher]]
        RBAC[[RBAC Manager]]
    end

    AuthDB[(Auth DB)]

    Controller --> ServiceLayer
    ServiceLayer --> UserRepo
    ServiceLayer --> RoleRepo
    ServiceLayer --> JWTProvider
    ServiceLayer --> PasswordHasher
    ServiceLayer --> RBAC

    UserRepo --> AuthDB
    RoleRepo --> AuthDB
```

---

# 🧩 Level 3 – Resource & Booking Service

> رزرو، جلوگیری از Overbooking، Lock Manager

```mermaid
flowchart TB
    subgraph BookingService ["Booking Service"]
        BookingController[[Booking Controller]]
        ReservationManager[[Reservation Manager]]
        AvailabilityChecker[[Availability Checker]]
        BookingRepo[(Booking Repo)]
        ResourceRepo[(Resource Repo)]
        LockManager[[Distributed Lock Manager]]
        EventPublisher[[Event Publisher]]
        NotificationClient[[Notification Client]]
    end

    Cache[(Redis Cache)]
    BookingDB[(Booking DB)]
    MessageBroker[(RabbitMQ)]

    BookingController --> ReservationManager
    ReservationManager --> AvailabilityChecker
    ReservationManager --> LockManager
    ReservationManager --> BookingRepo
    ReservationManager --> EventPublisher
    ReservationManager --> NotificationClient
    AvailabilityChecker --> Cache
    BookingRepo --> BookingDB
    ResourceRepo --> BookingDB
    EventPublisher --> MessageBroker
```

---

# 🧩 Level 3 — Marketplace Service

> فرآیند خرید با Saga + جبران (Compensation)

```mermaid
flowchart TB

    subgraph MarketplaceService ["Marketplace Service - Saga Orchestrator"]

        MarketplaceController[[Marketplace Controller]]
        OrderManager[[Order Manager]]
        SagaOrchestrator[[Saga Orchestrator]]
        InventoryChecker[[Inventory Checker]]
        PaymentClient[[Payment Client]]
        OrderRepo[(Order Repo)]
        ProductRepo[(Product Repo)]
        EventPublisher[[Event Publisher]]
        EventConsumer[[Event Consumer]]
        CompensationManager[[Compensation Manager]]

    end

    OrderDB[(Order DB)]
    ProductDB[(Product DB)]
    MessageBroker[(RabbitMQ)]
    IdempotencyStore[(Idempotency Store)]

    MarketplaceController --> OrderManager
    OrderManager --> InventoryChecker
    OrderManager --> PaymentClient
    OrderManager --> SagaOrchestrator
    OrderManager --> OrderRepo

    InventoryChecker --> ProductRepo
    ProductRepo --> ProductDB
    OrderRepo --> OrderDB

    SagaOrchestrator --> EventPublisher
    EventPublisher --> MessageBroker
    MessageBroker --> EventConsumer
    EventConsumer --> SagaOrchestrator

    SagaOrchestrator --> CompensationManager
    CompensationManager --> OrderRepo
    CompensationManager --> ProductRepo

    OrderManager --> IdempotencyStore
```

---

# 🧩 Level 3 — Exam Service

> شروع آزمون + Circuit Breaker + اعلان

```mermaid
flowchart TB

    subgraph ExamService ["Exam Service - Online Exams"]

        ExamController[[Exam Controller]]
        ExamManager[[Exam Manager]]
        ExamScheduler[[Exam Scheduler]]
        QuestionRepo[(Question Repo)]
        ExamRepo[(Exam Repo)]
        ResultRepo[(Result Repo)]
        NotificationClient[[Notification Client]]
        CB[[Circuit Breaker]]
        EventPublisher[[Event Publisher]]
        EventConsumer[[Event Consumer]]

    end

    ExamDB[(Exam DB)]
    QuestionDB[(Question DB)]
    ResultDB[(Result DB)]
    MessageBroker[(RabbitMQ)]
    Cache[(Redis Cache)]

    ExamController --> ExamManager
    ExamManager --> ExamRepo
    ExamManager --> QuestionRepo
    ExamManager --> ResultRepo
    ExamManager --> NotificationClient
    NotificationClient --> CB
    CB --> NotificationClient

    ExamManager --> Cache
    ExamManager --> EventPublisher
    EventPublisher --> MessageBroker
    MessageBroker --> EventConsumer

    ExamRepo --> ExamDB
    QuestionRepo --> QuestionDB
    ResultRepo --> ResultDB
```

---

# 🧩 Level 3 — IoT Service

> دریافت دادهٔ زنده — ردیابی شاتل — داشبورد

```mermaid
flowchart TB

    subgraph IoTService ["IoT Service - Live Sensors"]

        IoTController[[IoT Controller]]
        IoTIngestor[[Sensor Ingestor]]
        IoTProcessor[[Data Processor]]
        LocationTracker[[Shuttle Tracker]]
        DashboardService[[Dashboard Updater]]
        EventPublisher[[Event Publisher]]
        EventConsumer[[Event Consumer]]

    end

    IoTDB[(IoT DB)]
    Cache[(Redis Cache)]
    MessageBroker[(RabbitMQ)]

    IoTController --> IoTIngestor
    IoTIngestor --> IoTProcessor
    IoTProcessor --> LocationTracker
    IoTProcessor --> DashboardService
    IoTProcessor --> IoTDB
    IoTProcessor --> Cache

    IoTProcessor --> EventPublisher
    EventPublisher --> MessageBroker
    MessageBroker --> EventConsumer
```

---

# 🎉 فاز ۱ – کاملاً تکمیل شد

تمام موارد موردنیاز طبق PDF درس:

✔ Level 1
✔ Level 2
✔ Level 3 برای ۵ سرویس
✔ Event-driven flows
✔ Saga / Circuit Breaker
✔ Fully GitHub-renderable
✔ نسخه نهایی و یکپارچه

اگر بخواهی:

📌 نسخه **PDF آمادهٔ ارائه** بسازم
📌 یا نسخه **DOCX** برای تحویل
📌 یا **دو صفحه خلاصه Executive Summary**

فقط بگو:
**PDF می‌خوام** یا **DOCX بساز**.
