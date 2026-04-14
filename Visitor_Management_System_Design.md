# Visitor Management System Design

## Overview
This document provides a comprehensive overview of the architecture, data flow, and database structure for the Visitor Management System (VMS).

---

## High-Level Design (HLD)

classDiagram
    class User {
        +String name
        +String email
        +String phone
        +String role
        +register()
        +login()
    }

    class Visitor {
        +String fullName
        +String contactInfo
        +String companyName
        +String purposeOfVisit
        +DateTime checkInTime
        +DateTime checkOutTime
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
        +ObjectId visitorId
        +ObjectId hostId
        +Enum status
        +DateTime visitDate
        +submitRequest()
        +updateStatus()
    }

    class VisitorPass {
        +ObjectId requestId
        +String qrCode
        +DateTime validFrom
        +DateTime validTill
        +generatePass()
        +expirePass()
    }

    class AuditLog {
        +String action
        +ObjectId userId
        +DateTime timestamp
        +logEvent()
    }

    class VisitorController {
        +registerVisitor()
        +checkInVisitor()
        +checkOutVisitor()
        +getVisitorDetails()
    }

    class ApprovalController {
        +createVisitRequest()
        +respondToApproval()
        +preApproveVisitor()
    }

    class WorkerService {
        +sendNotification()
        +generateQRCode()
        +storeAuditTrail()
    }

    User "1" -- "0..N" VisitRequest : creates / manages
    HostEmployee "1" -- "0..N" VisitRequest : approves
    Visitor "1" -- "0..N" VisitRequest : belongs to
    VisitRequest "1" -- "0..1" VisitorPass : generates
    User "1" -- "0..N" AuditLog : produces
    VisitorController ..> Visitor : interacts
    ApprovalController ..> VisitRequest : manages
    WorkerService ..> VisitorPass : generates
    WorkerService ..> AuditLog : records

---

## Low-Level Design (LLD) - Class Diagram

- User: name, email, role
- Visitor: details, photo, check-in/out
- HostEmployee: approval actions
- VisitRequest: status, timestamps
- VisitorPass: QR, validity
- AuditLog: actions tracking

---

## ER Diagram (Entities)

- Users
- HostEmployees
- Visitors
- VisitRequests
- VisitorPasses
- AuditLogs

---

## Table Definitions

Users: authentication  
Visitors: visitor info  
VisitRequests: approval flow  
VisitorPasses: QR-based access  
AuditLogs: system tracking  

---

## Data Flow

1. Visitor registers  
2. Request created (PENDING)  
3. Notification sent  
4. Host approves/rejects  
5. Pass generated  
6. Check-in & check-out  
7. Logs stored  

---

## Caching

- Cache visitor + employee data  
- Faster QR validation  
- Reduces DB load  

---

## Authentication

- Role-based access (Admin, Guard, Employee)  
- Secure login (JWT / sessions)  

---

## Reliability

- Backup & recovery  
- Retry failed notifications  
- Error handling  

---

## Trade-offs

- Security vs speed  
- Caching vs freshness  

---

## Conclusion

System improves security, efficiency, and scalability.
