# 🛫 FlightOps Interview Guide - Baggage Handling & Tracking Module

**Your Name's Project Interview Preparation - Complete A to Z Guide**

---

## 📋 TABLE OF CONTENTS
1. [Project Overview](#project-overview)
2. [Your Module Deep Dive](#your-module-deep-dive)
3. [Complete API Guide](#complete-api-guide)
4. [Login & Authentication Flow](#login--authentication-flow)
5. [Real-World Workflow Example](#real-world-workflow-example)
6. [Interview Q&A](#interview-qa)
7. [Key Concepts Explained Simply](#key-concepts-explained-simply)

---

## 🚀 PROJECT OVERVIEW

### What is FlightOps?
FlightOps is a **Ground Operations Management System** that helps airports manage everything that happens on the ground when flights arrive and depart.

### Key Features:
- ✈️ **Flight Management** - Track flight schedules, status
- 👥 **Passenger Services** - Handle special assistance, check-ins
- 🛄 **Baggage Handling** - Your module! Track all baggage movement
- 🚗 **Ground Equipment** - Manage GSE (food trucks, tow vehicles, etc.)
- 🔧 **Maintenance Tracking** - Equipment service schedules
- 📊 **Turnaround Planning** - Coordinate activities between flights
- 🔐 **Security & Audit** - JWT authentication, audit logs

### Technology Stack:
```
Frontend: OpenAPI/Swagger (API documentation)
Backend: Spring Boot 4.0.6 (Java 21)
Database: MySQL 9.3
Security: JWT Tokens + Role-Based Access
Build: Maven
Logging: SLF4J (Logger Logging Framework)
```

---

## 🎯 YOUR MODULE DEEP DIVE: BAGGAGE HANDLING & TRACKING

### What Does Your Module Do?

Your module handles **THREE MAIN RESPONSIBILITIES**:

#### 1️⃣ **Baggage Operations** - Track Movement
- **Inbound**: Baggage arriving on flights being unloaded
- **Outbound**: Baggage departing on flights being loaded
- **Purpose**: Count total bags, track processed bags, detect discrepancies

#### 2️⃣ **Mishandled Baggage** - Report Problems
- Lost baggage
- Delayed baggage
- Damaged baggage
- Content theft/pilferage

#### 3️⃣ **Discrepancy Management** - Resolve Mismatches
- If expected ≠ processed → Mark as "Discrepancy"
- Alert supervisors automatically
- Prevent operation closure until resolved

---

### 📊 Data Models (Database Tables)

#### Table 1: `baggage_operations`
```
operationId (UUID)           → Unique ID for each baggage operation
flightId (String)            → Which flight this operation is for
direction (Enum)             → Inbound or Outbound
totalBagsExpected (Integer)  → How many bags should be on flight
totalBagsProcessed (Integer) → How many bags actually processed
operator (User)              → Which RampOfficer is handling this
startTime (DateTime)         → When operation started
endTime (DateTime)           → When operation ended (null if ongoing)
status (Enum)                → InProgress / Completed / Discrepancy
```

**Example Row**:
```
operationId: "op-12345-uuid"
flightId: "FL001"
direction: "Inbound"
totalBagsExpected: 200
totalBagsProcessed: 195
operator: "John Smith (RampOfficer)"
status: "Discrepancy" ⚠️ (5 bags missing!)
```

#### Table 2: `mishandled_baggage`
```
mishandleId (UUID)       → Unique incident ID
flightId (String)        → Which flight
passengerName (String)   → Passenger who reported issue
bagTagNumber (String)    → Unique baggage tag
mishandleType (Enum)     → Lost / Delayed / Damaged / PilferedContent
reportedDate (DateTime)  → When was it reported
status (Enum)            → Reported → Traced → Recovered → Claimed (or ClosedUnresolved)
```

**Example Row**:
```
mishandleId: "mh-67890-uuid"
flightId: "FL001"
passengerName: "Alice Johnson"
bagTagNumber: "BA-2024-12345"
mishandleType: "Lost"
status: "Traced" (currently being investigated)
```

---

### 🔐 User Roles & Permissions

Who can do what in your module?

| Role | Can Create Ops | Can Update Ops | Can Report Mishandled | Can View All | Purpose |
|------|---|---|---|---|---|
| **RampOfficer** | ✅ | ✅ | ✅ | ✅ | Hands-on baggage handling |
| **GroundSupervisor** | ❌ | ❌ | ❌ | ✅ | Oversee & resolve discrepancies |
| **PassengerAgent** | ❌ | ❌ | ❌ | ✅ Limited | Lookup baggage status for customers |
| **Admin** | ✅ | ✅ | ✅ | ✅ | Full system access |

---

### ✅ Status Enums Explained

#### For BaggageOperation:
```
InProgress → Operation is ongoing, bags being processed
Completed  → All bags processed, counts match ✅
Discrepancy → Expected ≠ Processed, investigation needed ⚠️
```

#### For MishandledBaggage:
```
Reported         → Just reported, hasn't been investigated
Traced           → Supervisor found where bag went
Recovered        → Bag found, back in system
Claimed          → Passenger got their bag ✅
ClosedUnresolved → Couldn't find bag, case closed ❌
```

#### For Baggage Movement:
```
Inbound  → Bags arriving on incoming flight (unloading)
Outbound → Bags departing on outgoing flight (loading)
```

---

## 🔌 COMPLETE API GUIDE

### Base Configuration
```
Base URL:        http://localhost:8080
Authentication:  JWT Bearer Token (required for all endpoints)
Response Format: JSON
```

### Header Template
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

---

### 📍 API ENDPOINT #1: Create Baggage Operation

**When**: Ramp officer starts handling baggage for a flight
**Endpoint**: `POST /api/baggage-ops`
**Who Can Use**: RampOfficer role only
**What It Does**: Initialize a baggage handling session

```bash
curl -X POST http://localhost:8080/api/baggage-ops \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "flightId": "FL001",
    "direction": "Inbound",
    "totalBagsExpected": 200,
    "startTime": "2024-06-16T10:00:00"
  }'
```

**Success Response** (Status 201):
```json
{
  "success": true,
  "message": "Baggage operation created",
  "data": {
    "operationId": "op-abc123-def456",
    "flightId": "FL001",
    "flightNumber": "BA101",
    "direction": "Inbound",
    "totalBagsExpected": 200,
    "totalBagsProcessed": 0,
    "discrepancy": 200,
    "operatorId": "user-xyz789",
    "operatorName": "John Smith",
    "startTime": "2024-06-16T10:00:00",
    "endTime": null,
    "status": "InProgress"
  }
}
```

**Error Response** (Status 409 - Conflict):
```json
{
  "success": false,
  "message": "A Inbound baggage operation already exists for flight BA101"
}
```
*Why?* Can't have 2 inbound operations for same flight

---

### 📍 API ENDPOINT #2: Get All Baggage Operations

**When**: Supervisor wants to see all ongoing/completed operations
**Endpoint**: `GET /api/baggage-ops`
**Who Can Use**: RampOfficer, GroundSupervisor, Admin
**What It Does**: List all baggage operations (sorted by newest first)

```bash
curl -X GET http://localhost:8080/api/baggage-ops \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response** (Status 200):
```json
{
  "success": true,
  "message": "Baggage operations fetched",
  "data": [
    {
      "operationId": "op-abc123",
      "flightNumber": "BA101",
      "direction": "Inbound",
      "totalBagsExpected": 200,
      "totalBagsProcessed": 200,
      "discrepancy": 0,
      "status": "Completed",
      "endTime": "2024-06-16T11:30:00"
    },
    {
      "operationId": "op-def456",
      "flightNumber": "BA102",
      "direction": "Outbound",
      "totalBagsExpected": 150,
      "totalBagsProcessed": 148,
      "discrepancy": 2,
      "status": "Discrepancy",
      "endTime": "2024-06-16T12:00:00"
    }
  ]
}
```

---

### 📍 API ENDPOINT #3: Get Operations for Specific Flight

**When**: You want to see baggage history for one flight
**Endpoint**: `GET /api/baggage-ops/flight/{flightId}`
**Who Can Use**: Anyone (no role check)
**What It Does**: List all baggage ops (inbound + outbound) for a flight

```bash
curl -X GET http://localhost:8080/api/baggage-ops/flight/FL001 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response** (Status 200):
```json
{
  "success": true,
  "message": "Baggage operations for flight fetched",
  "data": [
    {
      "operationId": "op-inbound-123",
      "flightNumber": "BA101",
      "direction": "Inbound",
      "status": "Completed",
      "totalBagsProcessed": 200
    },
    {
      "operationId": "op-outbound-456",
      "flightNumber": "BA101",
      "direction": "Outbound",
      "status": "InProgress",
      "totalBagsProcessed": 85
    }
  ]
}
```

---

### 📍 API ENDPOINT #4: Update Baggage Count

**When**: Ramp officer scans bags and updates count mid-operation
**Endpoint**: `PATCH /api/baggage-ops/{operationId}/count`
**Who Can Use**: RampOfficer role only
**What It Does**: Update the number of bags processed so far

```bash
curl -X PATCH http://localhost:8080/api/baggage-ops/op-abc123/count \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "totalBagsProcessed": 75
  }'
```

**Success Response** (Status 200):
```json
{
  "success": true,
  "message": "Bag count updated",
  "data": {
    "operationId": "op-abc123",
    "flightNumber": "BA101",
    "totalBagsExpected": 200,
    "totalBagsProcessed": 75,
    "discrepancy": 125,
    "status": "InProgress"
  }
}
```

**Error Response** (Status 400):
```json
{
  "success": false,
  "message": "Bags processed (250) cannot exceed bags expected (200)"
}
```
*Why?* Can't process more bags than expected

---

### 📍 API ENDPOINT #5: Complete Baggage Operation

**When**: Ramp officer finishes handling baggage for a flight
**Endpoint**: `PATCH /api/baggage-ops/{operationId}/complete`
**Who Can Use**: RampOfficer role only
**What It Does**: End the operation and check for discrepancies

```bash
curl -X PATCH http://localhost:8080/api/baggage-ops/op-abc123/complete \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Scenario A: Perfect Match** (Status 200):
```json
{
  "success": true,
  "message": "Baggage operation completed",
  "data": {
    "operationId": "op-abc123",
    "status": "Completed",
    "totalBagsExpected": 200,
    "totalBagsProcessed": 200,
    "discrepancy": 0,
    "endTime": "2024-06-16T11:30:00"
  }
}
```
✅ Perfect! No issues

**Scenario B: Discrepancy Detected** (Status 200):
```json
{
  "success": true,
  "message": "Baggage operation completed",
  "data": {
    "operationId": "op-abc123",
    "status": "Discrepancy",
    "totalBagsExpected": 200,
    "totalBagsProcessed": 195,
    "discrepancy": 5,
    "endTime": "2024-06-16T11:30:00"
  }
}
```
⚠️ **What happens**: 
- Status changed to "Discrepancy"
- Automatic notifications sent to GroundSupervisor and RampOfficer
- Investigation must happen before case closes

---

### 📍 API ENDPOINT #6: Report Mishandled Baggage

**When**: Ramp officer discovers a bag is lost, damaged, or delayed
**Endpoint**: `POST /api/mishandled`
**Who Can Use**: RampOfficer role only
**What It Does**: Create incident report for problematic baggage

```bash
curl -X POST http://localhost:8080/api/mishandled \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "flightId": "FL001",
    "passengerName": "Alice Johnson",
    "bagTagNumber": "BA-2024-12345",
    "mishandleType": "Damaged"
  }'
```

**Types You Can Report**:
- `Lost` - Bag can't be found
- `Delayed` - Bag arrived late
- `Damaged` - Bag has physical damage
- `PilferedContent` - Items missing from bag

**Success Response** (Status 201):
```json
{
  "success": true,
  "message": "Mishandled baggage reported",
  "data": {
    "mishandleId": "mh-xyz789-abc123",
    "flightId": "FL001",
    "flightNumber": "BA101",
    "passengerName": "Alice Johnson",
    "bagTagNumber": "BA-2024-12345",
    "mishandleType": "Damaged",
    "reportedDate": "2024-06-16T11:50:00",
    "status": "Reported"
  }
}
```

**Error Response** (Status 409):
```json
{
  "success": false,
  "message": "Bag tag BA-2024-12345 is already reported"
}
```
*Why?* Can't report same bag twice

---

### 📍 API ENDPOINT #7: Get All Mishandled Records

**When**: Supervisor wants to see all baggage issues
**Endpoint**: `GET /api/mishandled`
**Who Can Use**: RampOfficer, GroundSupervisor, Admin
**What It Does**: List all mishandled baggage incidents

```bash
curl -X GET http://localhost:8080/api/mishandled \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response** (Status 200):
```json
{
  "success": true,
  "message": "Mishandled bags fetched",
  "data": [
    {
      "mishandleId": "mh-123",
      "flightNumber": "BA101",
      "passengerName": "Alice Johnson",
      "bagTagNumber": "BA-2024-12345",
      "mishandleType": "Damaged",
      "status": "Reported",
      "reportedDate": "2024-06-16T11:50:00"
    },
    {
      "mishandleId": "mh-456",
      "flightNumber": "BA102",
      "passengerName": "Bob Smith",
      "bagTagNumber": "BA-2024-67890",
      "mishandleType": "Lost",
      "status": "Traced",
      "reportedDate": "2024-06-16T12:10:00"
    }
  ]
}
```

---

### 📍 API ENDPOINT #8: Query Specific Mishandled Baggage

**When**: Passenger agent helps customer find their bag status
**Endpoint**: `GET /api/mishandled/{bagTag}`
**Who Can Use**: RampOfficer, PassengerAgent
**What It Does**: Look up one specific baggage claim

```bash
curl -X GET http://localhost:8080/api/mishandled/BA-2024-12345 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Response** (Status 200):
```json
{
  "success": true,
  "message": "Mishandled bag fetched",
  "data": {
    "mishandleId": "mh-123",
    "flightNumber": "BA101",
    "passengerName": "Alice Johnson",
    "bagTagNumber": "BA-2024-12345",
    "mishandleType": "Damaged",
    "reportedDate": "2024-06-16T11:50:00",
    "status": "Recovered"
  }
}
```

---

### 📍 API ENDPOINT #9: Update Mishandled Baggage Status

**When**: Supervisor updates investigation progress
**Endpoint**: `PATCH /api/mishandled/{mishandleId}/status`
**Who Can Use**: RampOfficer role only
**What It Does**: Move case through investigation workflow

```bash
curl -X PATCH http://localhost:8080/api/mishandled/mh-xyz789/status \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "Traced"
  }'
```

**Status Workflow** (like a checklist):
```
1. Reported      ← Initial report
   ↓
2. Traced        ← Found where bag went
   ↓
3. Recovered     ← Bag recovered/fixed
   ↓
4. Claimed       ← Passenger got their bag ✅

OR

4. ClosedUnresolved ← Couldn't resolve ❌
```

**Example: Following the workflow**
```
# Initially reported
POST /api/mishandled
→ status: "Reported"

# Supervisor traced it
PATCH /api/mishandled/mh-123/status
→ status: "Traced"

# Bag was found and fixed
PATCH /api/mishandled/mh-123/status
→ status: "Recovered"

# Passenger claimed their bag
PATCH /api/mishandled/mh-123/status
→ status: "Claimed" ✅
```

---

## 🔐 LOGIN & AUTHENTICATION FLOW

### Step 1: Create Test Users (Admin does this once)
Different roles for different access levels:
```
Email: john.ramp@flightops.com     Role: RampOfficer (your primary role)
Email: alice.supervisor@flightops.com Role: GroundSupervisor
Email: bob.agent@flightops.com     Role: PassengerAgent
```

### Step 2: Login to Get JWT Token
**Endpoint**: `POST /api/auth/login`

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.ramp@flightops.com",
    "password": "YourPassword123!"
  }'
```

**Response** (Status 200):
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJqb2huLnJhbXBAZmxpZ2h0b3BzLmNvbSIsImlhdCI6MTcxODU0MjAwMCwiZXhwIjoxNzE4NjI4NDAwfQ.abc123def456",
  "message": "Login successful"
}
```

### Step 3: Use Token in Every Request
```bash
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Token Lifespan
- 🟢 Valid for: ~24 hours
- 🔴 After 24 hours: Need to login again

---

## 🎬 REAL-WORLD WORKFLOW EXAMPLE

### Scenario: Flight BA101 Arrives with 200 Bags

#### 🕐 10:00 AM - Flight Lands

**Step 1: RampOfficer logs in**
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -d '{"email":"john.ramp@flightops.com","password":"..."}'
```
Response: `token: eyJhbGc...`

**Step 2: Create baggage operation**
```bash
curl -X POST http://localhost:8080/api/baggage-ops \
  -H "Authorization: Bearer eyJhbGc..." \
  -d '{
    "flightId": "FL001",
    "direction": "Inbound",
    "totalBagsExpected": 200,
    "startTime": "2024-06-16T10:00:00"
  }'
```
✅ Response: Operation created, status = "InProgress"

#### 🕐 10:30 AM - Unloading in Progress

**Step 3: Update count - 1st batch (50 bags scanned)**
```bash
curl -X PATCH http://localhost:8080/api/baggage-ops/op-abc123/count \
  -d '{"totalBagsProcessed": 50}'
```
✅ Progress: 50/200 (150 remaining)

**Step 4: Update count - 2nd batch (100 bags total)**
```bash
curl -X PATCH http://localhost:8080/api/baggage-ops/op-abc123/count \
  -d '{"totalBagsProcessed": 100}'
```
✅ Progress: 100/200 (100 remaining)

#### 🕐 11:00 AM - Continue Unloading

**Step 5: Update count - 3rd batch (150 bags total)**
```bash
curl -X PATCH http://localhost:8080/api/baggage-ops/op-abc123/count \
  -d '{"totalBagsProcessed": 150}'
```
✅ Progress: 150/200 (50 remaining)

#### 🕐 11:30 AM - Complete Unloading BUT...

**Problem**: Only 195 bags unloaded (5 missing!)

**Step 6: Complete operation**
```bash
curl -X PATCH http://localhost:8080/api/baggage-ops/op-abc123/complete \
  -d '{"totalBagsProcessed": 195}'
```

**Response**:
```json
{
  "status": "Discrepancy",
  "discrepancy": 5,
  "message": "System auto-notified supervisors about 5 missing bags"
}
```

⚠️ **What Happens Automatically**:
- Status changes to "Discrepancy"
- Email/notification sent to Alice (GroundSupervisor)
- Alert: "5 bags unaccounted for on BA101 - Inbound"
- Supervisor investigates to find the missing bags

#### 🕐 11:45 AM - Discovery & Report

**Found**: 3 bags damaged, 2 bags delayed from cargo

**Step 7: Report first damaged bag**
```bash
curl -X POST http://localhost:8080/api/mishandled \
  -d '{
    "flightId": "FL001",
    "passengerName": "John Doe",
    "bagTagNumber": "BA-2024-11111",
    "mishandleType": "Damaged"
  }'
```
✅ Incident ID: mh-111

**Step 8: Report second damaged bag**
```bash
curl -X POST http://localhost:8080/api/mishandled \
  -d '{
    "flightId": "FL001",
    "passengerName": "Jane Smith",
    "bagTagNumber": "BA-2024-22222",
    "mishandleType": "Damaged"
  }'
```
✅ Incident ID: mh-222

**Step 9: Report two delayed bags**
```bash
curl -X POST http://localhost:8080/api/mishandled \
  -d '{
    "flightId": "FL001",
    "passengerName": "Bob Johnson",
    "bagTagNumber": "BA-2024-33333",
    "mishandleType": "Delayed"
  }'
```
✅ Incident ID: mh-333

#### 🕐 12:00 PM - Investigation Progress

**Step 10: Supervisor updates status - Bag found**
```bash
curl -X PATCH http://localhost:8080/api/mishandled/mh-111/status \
  -d '{"status": "Traced"}'
```

**Step 11: Bag repaired**
```bash
curl -X PATCH http://localhost:8080/api/mishandled/mh-111/status \
  -d '{"status": "Recovered"}'
```

**Step 12: Passenger claims bag**
```bash
curl -X PATCH http://localhost:8080/api/mishandled/mh-111/status \
  -d '{"status": "Claimed"}'
```
✅ Case closed successfully!

---

## ❓ INTERVIEW Q&A

### Q1: "Describe your baggage module in simple terms"

**Good Answer**:
> "The baggage module has two main parts. First, BaggageOperations tracks how many bags arrive or depart on a flight - we expect 200 bags, and check how many we actually process. If they don't match, we flag it as a Discrepancy and alert supervisors. Second, MishandledBaggage tracks individual problematic bags - lost, damaged, delayed, or stolen items. When we report a mishandled bag, it goes through a workflow: Reported → Traced → Recovered → Claimed. This helps us keep track of customer issues."

---

### Q2: "How does your module handle discrepancies?"

**Good Answer**:
> "When a baggage operation ends, if totalBagsProcessed doesn't equal totalBagsExpected, we automatically:
> 1. Mark the operation status as 'Discrepancy'
> 2. Send notifications to GroundSupervisors and RampOfficers
> 3. They can then investigate to find the missing bags
> 4. Once resolved, they update mishandled records accordingly"

---

### Q3: "What are the different user roles and what can they do?"

**Good Answer**:
> "We have 4 main roles:
> - **RampOfficer**: Can create operations, update bag counts, and report mishandled bags
> - **GroundSupervisor**: Can view all operations, see discrepancies, receive alerts
> - **PassengerAgent**: Can look up specific bags to help customers track claims
> - **Admin**: Full access to everything"

---

### Q4: "Walk me through reporting a lost bag"

**Good Answer**:
> "1. RampOfficer discovers a bag is missing during unloading
> 2. They call POST /api/mishandled with:
>    - Flight ID
>    - Passenger name
>    - Bag tag number
>    - Type: 'Lost'
> 3. System creates incident record, status = 'Reported'
> 4. Supervisor gets notified
> 5. They investigate and update status to 'Traced'
> 6. If bag found, mark 'Recovered', then 'Claimed'
> 7. If bag not found, mark 'ClosedUnresolved'"

---

### Q5: "What prevents duplicate records?"

**Good Answer**:
> "We have two main uniqueness constraints:
> 1. Can't create two 'Inbound' operations for the same flight (we check: does this flight + direction already have an operation?)
> 2. Can't report the same bag tag twice (we check: does this bag tag already exist in mishandled records?)
> This prevents data inconsistencies and confusion."

---

### Q6: "How do you use role-based access control?"

**Good Answer**:
> "We use Spring Security with @PreAuthorize annotations. For example:
> - @PreAuthorize('hasRole(\"RampOfficer\")') on create operation
> - @PreAuthorize('hasAnyRole(\"RampOfficer\", \"GroundSupervisor\", \"Admin\")') on get all operations
> - @PreAuthorize('hasAnyRole(\"RampOfficer\", \"PassengerAgent\")') on lookup mishandled bag
> If user doesn't have the role, system returns 403 Forbidden"

---

### Q7: "What happens if a passenger reports their bag as damaged?"

**Good Answer**:
> "1. RampOfficer creates a mishandled baggage record with type='Damaged'
> 2. Status starts as 'Reported'
> 3. Supervisor is alerted via notification
> 4. Supervisor updates status to 'Traced' (investigated)
> 5. If damage confirmed, 'Recovered' (fixed or compensated)
> 6. Finally 'Claimed' when passenger receives resolution
> Each status update helps track the resolution process"

---

### Q8: "How does the system notify people about discrepancies?"

**Good Answer**:
> "When bagging operation completes with a discrepancy:
> 1. System calculates: expected - processed = discrepancy
> 2. If discrepancy > 0, operation status = 'Discrepancy'
> 3. System finds all users with roles: GroundSupervisor, RampOfficer
> 4. Sends each of them a notification with details:
>    - Flight number
>    - Direction (Inbound/Outbound)
>    - Number of bags missing
> 5. They receive alert and can investigate"

---

## 🎓 KEY CONCEPTS EXPLAINED SIMPLY

### Concept 1: JWT Tokens
**What**: A secure encrypted string that proves you're logged in
**Why**: Instead of sending your password every time, you send this token
**How Long**: Valid for ~24 hours, then you need to login again
**Analogy**: It's like a digital badge - proves you're an employee without carrying ID constantly

---

### Concept 2: Role-Based Access Control (RBAC)
**What**: Different users have different permissions based on their job title
**Why**: Security - you don't want a PassengerAgent creating operations
**How It Works**:
- Each user assigned a Role (RampOfficer, Supervisor, etc.)
- Each endpoint checks: "Does this user's role have permission?"
- If yes: ✅ Process request
- If no: ❌ Return 403 Forbidden

**Analogy**: Like a building with different access cards - electrician can't go in the vault

---

### Concept 3: Transactional Operations
**What**: Database operations that either fully succeed or fully fail
**Why**: Prevents partial updates that corrupt data
**Example**: 
- You update operation status to "Discrepancy"
- AND send notification
- If notification fails → whole thing rolls back
- No operation status change without notification

**Analogy**: Like online banking - money transfers all-or-nothing

---

### Concept 4: RESTful API Design
**What**: Organizing API endpoints around resources and HTTP methods

| HTTP Verb | Means | Example |
|-----------|-------|---------|
| POST | Create new | Create baggage operation |
| GET | Read/retrieve | Get all operations |
| PATCH | Partial update | Update bag count |
| DELETE | Remove | (not used in baggage module) |

**Analogy**: Like CRUD operations (Create, Read, Update, Delete)

---

### Concept 5: Status Enums
**What**: Predefined list of possible values (not free text)
**Why**: 
- Prevents typos (InProgress vs In Progress vs Inprogress)
- Database knows exact values
- System can act on specific statuses

**Example in your code**:
```
Direction: {Inbound, Outbound}    ← Only 2 options
OperationStatus: {InProgress, Completed, Discrepancy} ← Only 3 options
```

**Analogy**: Traffic lights can only be Red/Yellow/Green, never "Pinkish"

---

### Concept 6: Entities vs DTOs
**What**:
- **Entity**: Database table (BaggageOperation, MishandledBaggage)
- **DTO**: Data Transfer Object - what you send/receive in API

**Why**: DTOs hide internal database structure from API users
**Example**:
```
Entity has: password, internalId, databaseTimestamp
DTO only has: name, role, visibleId
```

**Analogy**: Hotel database (Entity) has private info, but guest (DTO) only sees room number

---

### Concept 7: Soft vs Hard Failures
**Soft Failure** (Discrepancy detection):
- Expected 200, got 195 → Not an error, but flagged as issue
- Operation still completes, status = "Discrepancy"
- Triggers investigation workflow

**Hard Failure** (Validation error):
- Processed count > Expected count → Rejects request
- Returns 400 Bad Request
- Request fails completely

**Analogy**: Soft = warning light on dashboard, Hard = engine doesn't start

---

## 💡 TIPS FOR YOUR INTERVIEW

### DO's ✅
- Explain your module **in simple terms** - not everyone knows Spring Boot
- Use **real examples** - "On flight BA101, we expect 200 bags but only get 195..."
- Show **understanding of the flow** - Create → Update → Complete → Report Issues
- Mention **error handling** - "We prevent duplicate operations and duplicate bag reports"
- Talk about **notifications** - "System automatically alerts supervisors of discrepancies"
- Reference **roles** - "Different users have different permissions for security"

### DONT's ❌
- Don't use **jargon without explaining** - "Entity-DTO mapping pattern..." → "We separate database tables from API responses"
- Don't get **too technical** - "We use Lombok annotations to reduce boilerplate" → mention if asked, otherwise skip
- Don't **memorize code** - Understand the logic, not syntax
- Don't **claim knowledge** you don't have - "Good question, let me think about that"
- Don't **ignore errors** - "What if someone sends bad data?" shows good thinking

---

## 📞 Quick Reference - API URLs for Interview

```
Authentication:
  POST /api/auth/login

Baggage Operations:
  POST /api/baggage-ops                           (Create)
  GET /api/baggage-ops                            (Get All)
  GET /api/baggage-ops/flight/{flightId}          (Filter by Flight)
  PATCH /api/baggage-ops/{id}/count               (Update Count)
  PATCH /api/baggage-ops/{id}/complete            (Complete Operation)

Mishandled Baggage:
  POST /api/mishandled                            (Report)
  GET /api/mishandled                             (Get All)
  GET /api/mishandled/{bagTag}                    (Lookup One)
  PATCH /api/mishandled/{id}/status               (Update Status)
```

---

## 🎯 30-Second Elevator Pitch for Your Module

> "I built the baggage handling and tracking system for FlightOps. When a flight arrives, we create a baggage operation to track how many bags we expect versus how many we actually process. If there's a mismatch, the system automatically alerts supervisors. We also have a mishandled baggage system for reporting lost, damaged, or delayed bags - the system tracks them through an investigation workflow from reported → traced → recovered → claimed. It uses role-based access control, so RampOfficers can create and update operations, supervisors can investigate issues, and passenger agents can look up bag status for customers."

---

**Good luck with your interview! You've got this! 🚀**
