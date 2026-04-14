# Visitor Management System Design

This document provides a comprehensive overview of the architecture, data flow, and database structure for the Visitor Management System (VMS).

---

## High-Level Design (HLD)

This diagram illustrates the data flow from the frontend to the database, incorporating **Redis Cache and Workers** for asynchronous processing.

```mermaid
graph TD
    subgraph Client_Side [Client Side]
        FE[Frontend - Web Portal / Security Desk / Kiosk]
    end

    subgraph API_Layer [API Layer]
        LB[Load Balancer / Nginx]
        BE[Backend Application Servers]
    end

    subgraph Data_Storage [Data Storage]
        DB[(MySQL / MongoDB)]
        RC[(Redis Cache)]
        RQ[(Redis - Task Queue)]
    end

    subgraph Asynchronous_Workers [Asynchronous Workers]
        WR[Worker Processes]
        NS[Notification Service]
        QR[QR / Pass Generator]
        AL[Audit Logging Service]
    end

    FE -- HTTPS/REST --> LB
    LB --> BE
    BE -- CRUD Operations --> DB
    BE -- Cache Frequent Data --> RC
    BE -- Enqueue Tasks --> RQ
    RQ -- Pull Tasks --> WR
    WR -- Send Notifications --> NS
    WR -- Generate Pass --> QR
    WR -- Log Activity --> AL
    QR --> DB
    AL --> DB
    NS -- Notify Host/Guard --> FE
```

---

## Low-Level Design (LLD) - Class Diagram

The class diagram outlines the relationships between controllers, models, and core services.

```mermaid
classDiagram
    class User {
        +String name
        +String email
        +String password
        +String role
        +login()
        +register()
    }

    class Visitor {
        +String fullName
        +String contactInfo
        +String company
        +String purpose
        +capturePhoto()
        +registerVisitor()
    }

    class HostEmployee {
        +String employeeId
        +String department
        +approveVisitor()
        +rejectVisitor()
    }

    class VisitRequest {
        +ObjectId visitor_id
        +ObjectId host_id
        +Enum status
        +DateTime visit_time
        +submitRequest()
        +updateStatus()
    }

    class VisitorPass {
        +ObjectId request_id
        +String qr_code
        +DateTime valid_from
        +DateTime valid_till
        +generatePass()
    }

    class VisitorController {
        +registerVisitor()
        +checkIn()
        +checkOut()
    }

    class ApprovalController {
        +createRequest()
        +respondApproval()
        +preApprove()
    }

    class WorkerService {
        +sendNotification()
        +generateQR()
        +logActivity()
    }

    User "1" -- "0..N" VisitRequest : manages
    Visitor "1" -- "0..N" VisitRequest : creates
    HostEmployee "1" -- "0..N" VisitRequest : approves
    VisitRequest "1" -- "0..1" VisitorPass : generates
    VisitorController ..> Visitor : interacts
    ApprovalController ..> VisitRequest : manages
    WorkerService ..> VisitorPass : generates
```

---

## Entity-Relationship (ER) Diagram

A relational view of the database schemas and their constraints.

```mermaid
erDiagram
    USER ||--o{ VISIT_REQUEST : creates
    HOST_EMPLOYEE ||--o{ VISIT_REQUEST : approves
    VISITOR ||--o{ VISIT_REQUEST : submits
    VISIT_REQUEST ||--o| VISITOR_PASS : generates

    USER {
        ObjectId _id PK
        String name
        String email
        String password_hash
        String role
        Date createdAt
    }

    HOST_EMPLOYEE {
        ObjectId _id PK
        String employee_id
        String department
        String email
    }

    VISITOR {
        ObjectId _id PK
        String full_name
        String contact_info
        String company
        String purpose
        String photo_url
    }

    VISIT_REQUEST {
        ObjectId _id PK
        ObjectId visitor_id FK
        ObjectId host_id FK
        Enum status "PENDING, APPROVED, REJECTED"
        DateTime visit_time
        DateTime check_in
        DateTime check_out
    }

    VISITOR_PASS {
        ObjectId _id PK
        ObjectId request_id FK
        String qr_code
        DateTime valid_from
        DateTime valid_till
    }
```

---

## Table Definitions & Idea

| Table (Collection) | Description | Key Fields | Purpose |
|---|---|---|---|
| Users | System users (Admin, Guard) | _id, email, password_hash, role | Authentication & access |
| HostEmployees | Employees receiving visitors | _id, employee_id, department | Approval authority |
| Visitors | Visitor details | _id, full_name, contact_info, purpose | Identity tracking |
| VisitRequests | Approval workflow | _id, visitor_id, host_id, status | Core logic |
| VisitorPasses | QR-based entry pass | _id, request_id, qr_code | Entry validation |
| AuditLogs | System logs | _id, action, user_id, timestamp | Monitoring |

---

## Data Flow Breakdown (Visual Summary)

1. Frontend: Security guard enters visitor details  
2. API: VisitorController.registerVisitor handles request  
3. Database: Visitor stored  
4. Request: VisitRequest created with status PENDING  
5. Queue: Task sent to Redis  
6. Worker: Notifies host employee  
7. Approval: Host approves/rejects  
8. Pass: QR-based visitor pass generated  
9. Check-in: Visitor enters  
10. Check-out: Exit recorded  
11. Logs: Stored for audit  

---

## Caching

- Cache frequently accessed data such as employee details and active visitor passes  
- Improves response time  
- Reduces database load  
- Example: QR validation can be done via Redis instead of DB  

---

## Authentication & Authorization

- Role-based access:
  - Admin
  - Security Guard
  - Host Employee  

- Secure login using JWT / sessions  
- Ensures only authorized access  

---

## Reliability & Failure Handling

- Backup and recovery mechanisms  
- Retry failed notifications  
- Graceful error handling  
- Audit logging for debugging  

---

## Trade-offs

- Security vs Convenience  
- Performance vs Data Freshness (Caching)  
- Real-time processing vs system complexity  

---

## Conclusion

The Visitor Management System improves campus security, automates visitor tracking, and ensures efficient and scalable access control using modern system design principles.
