# 🏗️ FLIGHTOPS PROJECT ARCHITECTURE - Comprehensive Overview

**For understanding how your Baggage Module fits into the entire project**

---

## 📊 PROJECT STRUCTURE AT A GLANCE

```
FlightOps (Ground Operations Management System)
│
├─ Frontend Layer (OpenAPI/Swagger Documentation)
│
├─ API Layer (REST Controllers)
│  ├─ Authentication Controller (Login/Registration)
│  ├─ Flight Controller (Flight management)
│  ├─ Baggage Controller ⭐ (YOUR MODULE)
│  ├─ Passenger Controller (Passenger services)
│  ├─ Equipment Controller (GSE management)
│  └─ Other Controllers...
│
├─ Business Logic Layer (Services)
│  ├─ Authentication Service
│  ├─ Flight Service
│  ├─ Baggage Service ⭐ (YOUR MODULE)
│  ├─ Passenger Service
│  ├─ Equipment Service
│  ├─ Notification Service
│  └─ Other Services...
│
├─ Data Access Layer (Repositories)
│  ├─ User Repository
│  ├─ Flight Repository
│  ├─ Baggage Operation Repository ⭐ (YOUR MODULE)
│  ├─ Mishandled Baggage Repository ⭐ (YOUR MODULE)
│  └─ Other Repositories...
│
├─ Database Layer (MySQL)
│  ├─ Users Table
│  ├─ Flights Table
│  ├─ BaggageOperations Table ⭐ (YOUR MODULE)
│  ├─ MishandledBaggage Table ⭐ (YOUR MODULE)
│  └─ Other Tables...
│
├─ Security Layer
│  ├─ JWT Filter (intercepts requests)
│  ├─ Spring Security Config (role-based access)
│  └─ Password Encoder (BCrypt)
│
└─ Utility & Configuration
   ├─ OpenAPI/Swagger (API documentation)
   ├─ Custom Exceptions (error handling)
   ├─ DTOs (data transfer objects)
   └─ Enums (constants like roles, statuses)
```

---

## 🎯 YOUR MODULE IN THE BIG PICTURE

### What is FlightOps?
FlightOps is a **complete airport ground operations management system**. Think of it as:
- **Orchestrator** of everything that happens between flights
- **Coordination tool** for multiple ground handling teams
- **Real-time tracking** of assets, people, and processes
- **Notification hub** that alerts the right people at the right time

### Where Does Baggage Fit?
Your baggage module is a **critical operational piece** because:
1. **Baggage is customer-facing** - Passengers care about their bags
2. **Revenue impact** - Mishandled bags = compensation costs
3. **Operational efficiency** - Baggage delays can delay entire flight
4. **Integration point** - Connects with flights, passengers, notifications

### How It Connects to Other Modules

```
FLIGHT ARRIVES
    ↓
Flight Status Updated (Flight Module)
    ↓
Baggage Operation Created (Baggage Module) ⭐
    ↓
    ├─→ Real-time updates (Baggage Module)
    ├─→ Discrepancy Alert (Notification Module)
    └─→ Supervisor Investigation (User Module)
    
PASSENGER CLAIMS BAG
    ↓
Passenger Status Updated (Passenger Module)
    ↓
Baggage Status Closed (Baggage Module) ⭐
    ↓
Notification Sent (Notification Module)
```

---

## 🔧 TECHNICAL ARCHITECTURE EXPLAINED

### Layer 1: Controller Layer (What Users See)
```java
@RestController
public class BaggageController {
    // Endpoints users call
    POST /api/baggage-ops
    GET /api/baggage-ops
    PATCH /api/baggage-ops/{id}/complete
    POST /api/mishandled
    // ... etc
}
```
**Purpose**: Accept HTTP requests and return responses

---

### Layer 2: Service Layer (Business Logic)
```java
@Service
public class BaggageService {
    // Core logic
    createOperation()        // Validate and create
    updateCount()           // Update with validations
    completeOperation()     // Check for discrepancy
    reportMishandled()      // Create incident
    // ... etc
}
```
**Purpose**: Implement business rules and validations

---

### Layer 3: Repository Layer (Database Access)
```java
@Repository
public interface BaggageOperationRepository extends JpaRepository {
    // Query database
    findByFlight_FlightId()
    findByStatus()
    existsByFlight_FlightIdAndDirection()
}
```
**Purpose**: Communicate with MySQL database

---

### Layer 4: Entity Layer (Data Models)
```java
@Entity
@Table(name = "baggage_operations")
public class BaggageOperation {
    private String operationId;
    private Flight flight;
    private Direction direction;
    private Integer totalBagsExpected;
    // ... etc
}
```
**Purpose**: Represents database tables in Java

---

## 📊 DATA FLOW EXAMPLE: Creating a Baggage Operation

```
USER REQUEST
    │
    ├─ POST /api/baggage-ops
    ├─ Authorization: Bearer JWT_TOKEN
    ├─ Body: {flightId, direction, totalBagsExpected, ...}
    │
    ▼
[HTTP Layer - Spring Web]
    ├─ Parse request
    ├─ Extract JWT token
    ├─ Identify user role
    │
    ▼
[Security Filter - JWT Filter]
    ├─ Verify token is valid
    ├─ Check if expired
    ├─ Extract user details
    │
    ▼
[Controller Layer - BaggageController]
    ├─ Check @PreAuthorize("hasRole('RampOfficer')")
    ├─ Call service method
    │
    ▼
[Service Layer - BaggageService]
    ├─ Get Flight from flightService.findById(flightId)
    ├─ Check if operation already exists
    ├─ Get User from userRepository.findByEmail(email)
    ├─ Create BaggageOperation object
    ├─ Set status = "InProgress"
    │
    ▼
[Repository Layer - BaggageOperationRepository]
    ├─ Save object to database
    ├─ Return saved object with UUID generated
    │
    ▼
[Database Layer - MySQL]
    ├─ INSERT INTO baggage_operations (...)
    ├─ Return new record
    │
    ▼
[Service Layer - Back to service]
    ├─ Convert entity to DTO
    ├─ Return response object
    │
    ▼
[Controller Layer - Back to controller]
    ├─ Wrap response in ApiResponse
    ├─ Return with status 201 Created
    │
    ▼
USER RESPONSE
    └─ JSON with new operation details
```

---

## 🔐 Security Architecture

### Authentication Flow
```
User: "I want to access the system"
    ↓
Step 1: Login
    POST /api/auth/login
    Email: john.ramp@flightops.com
    Password: ****
    ↓
Step 2: System validates credentials
    Spring Security DaoAuthenticationProvider
    Uses BCryptPasswordEncoder to verify
    ↓
Step 3: If valid, generate JWT token
    Token contains: userId, email, roles, expiration
    ↓
Step 4: Return token to user
    Response: { token: "eyJhbGc..." }
    ↓
Step 5: User includes token in all requests
    Header: Authorization: Bearer eyJhbGc...
    ↓
Step 6: For each request
    JWT Filter intercepts
    Validates token signature
    Checks expiration
    Extracts user info
    ↓
Step 7: Spring Security checks @PreAuthorize
    @PreAuthorize("hasRole('RampOfficer')")
    User has role? YES → Allow ✅
    User has role? NO → 403 Forbidden ❌
```

### Authorization (Role-Based Access)
```
User logs in → Token issued with Role

Every endpoint checks:
    @PreAuthorize("hasRole('RampOfficer')")
    
Becomes:
    if (userToken.getRole() == "RampOfficer") {
        ✅ Allow access
    } else {
        ❌ Return 403 Forbidden
    }
```

---

## 📦 REQUEST/RESPONSE PATTERN

### Standard Request Format
```
curl -X POST http://localhost:8080/api/baggage-ops \
  -H "Authorization: Bearer eyJhbGc..." \
  -H "Content-Type: application/json" \
  -d '{
    "flightId": "FL001",
    "direction": "Inbound",
    "totalBagsExpected": 200,
    "startTime": "2024-06-16T10:00:00"
  }'
```

### Standard Response Format (All Endpoints)
```json
{
  "success": true,
  "message": "Operation created successfully",
  "data": {
    "operationId": "op-uuid-123",
    "flightId": "FL001",
    "direction": "Inbound",
    ...
  }
}
```

**Note**: Uses wrapper class `ApiResponse<T>` for consistency

### Error Response Format
```json
{
  "success": false,
  "message": "Error description",
  "error": "DetailedErrorInfo"
}
```

---

## 🧪 TESTING & VALIDATION

### What Gets Validated?

#### Input Validation (DTO Level)
```java
@Data
public class BaggageOperationRequest {
    @NotBlank(message = "Flight ID is required")
    private String flightId;
    
    @NotNull(message = "Direction is required")
    private Direction direction;
    
    @Positive(message = "Expected bags must be positive")
    private Integer totalBagsExpected;
}
```

#### Business Logic Validation (Service Level)
```java
// Check duplicate operation exists
if (baggageOperationRepository.existsByFlight_FlightIdAndDirection(...)) {
    throw new ConflictException("Operation already exists");
}

// Check processed count
if (request.getTotalBagsProcessed() > operation.getTotalBagsExpected()) {
    throw new BadRequestException("Can't process more than expected");
}
```

#### Authorization Validation (Controller Level)
```java
@PostMapping("/api/baggage-ops")
@PreAuthorize("hasRole('RampOfficer')")  // Only RampOfficer
public ResponseEntity<...> create(...) {
    // ...
}
```

---

## 🔔 NOTIFICATION SYSTEM INTEGRATION

When your module creates critical events, it notifies other users:

```java
// In BaggageService.completeOperation()
if (discrepancy != 0) {
    // Alert supervisors and ramp officers
    List.of(Role.GroundSupervisor, Role.RampOfficer).forEach(role ->
        userRepository.findByRole(role).forEach(user ->
            notificationService.sendNotification(
                user.getUserId(),
                "Baggage discrepancy on flight " + flight.getFlightNumber() + 
                ": " + discrepancy + " bags unaccounted",
                NotificationCategory.Baggage
            )
        )
    );
}
```

**What This Does**:
1. Find all supervisors and ramp officers
2. For each user, send a notification
3. Message includes: Flight number, bags missing, category
4. Users see alert in their dashboard/email

---

## 💾 DATABASE SCHEMA (Simplified)

### baggage_operations Table
```sql
CREATE TABLE baggage_operations (
    operation_id VARCHAR(36) PRIMARY KEY,
    flight_id VARCHAR(36) NOT NULL,
    direction VARCHAR(20),  -- Inbound or Outbound
    total_bags_expected INT,
    total_bags_processed INT,
    operator_id VARCHAR(36),
    start_time DATETIME,
    end_time DATETIME,
    status VARCHAR(20),  -- InProgress, Completed, Discrepancy
    FOREIGN KEY (flight_id) REFERENCES flights(flight_id),
    FOREIGN KEY (operator_id) REFERENCES users(user_id)
);
```

### mishandled_baggage Table
```sql
CREATE TABLE mishandled_baggage (
    mishandle_id VARCHAR(36) PRIMARY KEY,
    flight_id VARCHAR(36) NOT NULL,
    passenger_name VARCHAR(100),
    bag_tag_number VARCHAR(50) UNIQUE,
    mishandle_type VARCHAR(20),  -- Lost, Damaged, Delayed, PilferedContent
    reported_date DATETIME,
    status VARCHAR(20),  -- Reported, Traced, Recovered, Claimed, ClosedUnresolved
    FOREIGN KEY (flight_id) REFERENCES flights(flight_id)
);
```

---

## 🚀 DEPLOYMENT & RUNNING THE PROJECT

### Prerequisites
- Java 21 installed
- MySQL running (or database connection)
- Maven installed

### Build & Run
```bash
# Build
mvn clean install

# Run
mvn spring-boot:run

# Application starts on:
http://localhost:8080
```

### API Documentation
```
Swagger UI:    http://localhost:8080/swagger-ui.html
OpenAPI JSON:  http://localhost:8080/api-docs
```

---

## 🎯 INTERVIEW: HOW TO DESCRIBE PROJECT ARCHITECTURE

**If asked: "Describe your project architecture"**

Good Answer:
> "FlightOps is built using a layered architecture. We have a Controller layer that handles HTTP requests, a Service layer where I implement business logic, a Repository layer for database access, and Entities representing database tables. For the baggage module specifically, when a flight arrives, the RampOfficer creates a baggage operation to track expected and processed bags. The system uses Spring Security with JWT for authentication and role-based access control to ensure only authorized users can perform operations. If there's a discrepancy in bag counts, the system automatically notifies supervisors. The module integrates with the flight and notification modules to provide a seamless operational experience."

---

## 📈 SCALABILITY & PERFORMANCE

### Current Architecture Supports:
- ✅ Multiple concurrent flights
- ✅ Real-time baggage tracking
- ✅ Automatic notifications
- ✅ Role-based access control
- ✅ Audit logging (if implemented)

### How It Scales:
1. **Database indexing** on frequently searched columns
2. **Lazy loading** of related entities
3. **Pagination** for large data sets
4. **Transactional isolation** prevents race conditions

---

## 🔄 TYPICAL DAY OPERATIONS

**Morning: Flight BA101 Arrives**
```
08:00 - Flight scheduled
10:00 - Flight lands
10:02 - John (RampOfficer) creates baggage operation
10:05 - 50 bags scanned and counted
10:15 - 100 bags counted
10:25 - 195 bags counted (5 missing!)
10:30 - Operation completed → Discrepancy detected
10:31 - Alice (Supervisor) gets alert
10:35 - Investigation begins
11:00 - Found 3 damaged, 2 delayed
11:30 - All 5 accounted for → Cases closed
12:00 - Flight ready to turnaround
```

---

## 📚 KEY CONCEPTS IN SIMPLE TERMS

| Concept | In Simple Terms | Example |
|---------|-----------------|---------|
| **Entity** | Database table | BaggageOperation = one row per operation |
| **DTO** | What we send/receive | BaggageOperationRequest = input format |
| **Repository** | Database queries | Find all operations, check if exists |
| **Service** | Business logic | Validate, calculate discrepancy, save |
| **Controller** | API endpoints | POST /api/baggage-ops |
| **JWT Token** | Digital ID card | Proves you're logged in & your role |
| **Role** | Job title | RampOfficer, Supervisor |
| **Transactional** | All or nothing | Update status AND send notification together |

---

## 🎓 FOR YOUR INTERVIEW

### Know How to Explain:
1. ✅ What your module does
2. ✅ How it connects to other modules
3. ✅ The data flow (request → response)
4. ✅ Security & authorization
5. ✅ Error handling & validations
6. ✅ Notification system
7. ✅ Role-based access

### Practice Explaining With:
- API endpoints
- Real workflow examples
- Database relationships
- Security mechanisms
- Scalability considerations

### Confidence Booster Phrases:
- "The architecture follows REST principles..."
- "We use JPA for ORM and Spring Security for authorization..."
- "The service layer contains all business logic..."
- "Data flows from controller to service to repository..."
- "Notifications are sent through our notification module..."

---

**You've got a solid understanding of the project now! 🚀**

*Read this before your interview to refresh on architecture context!*
