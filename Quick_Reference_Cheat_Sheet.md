# 📌 QUICK REFERENCE - BAGGAGE MODULE CHEAT SHEET

**Print this and keep it with you during interview! ⏱️ Quick lookup (5 seconds)**

---

## 🎯 30-SECOND MODULE SUMMARY

*"The baggage module tracks both baggage movement and baggage problems. We create operations to count bags arriving/departing on flights, update counts in real-time, and if there's a mismatch, we flag it as a discrepancy and alert supervisors. We also have a mishandled baggage system for tracking lost, damaged, delayed, or stolen bags through an investigation workflow."*

---

## 📊 TWO MAIN ENTITIES

| BaggageOperation | MishandledBaggage |
|---|---|
| Tracks overall baggage counts | Tracks individual problem bags |
| One per flight per direction | One per problematic bag |
| Status: InProgress → Completed/Discrepancy | Status: Reported → Traced → Recovered → Claimed |
| Updates: count, completion | Updates: status only |
| Example: FL001 Inbound, 200 bags expected | Example: BA-12345 lost, needs investigation |

---

## 🔐 ROLES & WHAT THEY CAN DO

| Role | Create Op | Update Op | Report | View All |
|------|:-:|:-:|:-:|:-:|
| RampOfficer | ✅ | ✅ | ✅ | ✅ |
| GroundSupervisor | ❌ | ❌ | ❌ | ✅ |
| PassengerAgent | ❌ | ❌ | ❌ | 🔍 |
| Admin | ✅ | ✅ | ✅ | ✅ |

---

## 🔗 API ENDPOINTS - MEMORIZE THESE

### Baggage Operations
```
POST   /api/baggage-ops                      Create operation
GET    /api/baggage-ops                      Get all
GET    /api/baggage-ops/flight/{flightId}    Filter by flight
PATCH  /api/baggage-ops/{id}/count           Update processed count
PATCH  /api/baggage-ops/{id}/complete        End operation
```

### Mishandled Baggage
```
POST   /api/mishandled                       Report issue
GET    /api/mishandled                       Get all
GET    /api/mishandled/{bagTag}              Lookup one
PATCH  /api/mishandled/{id}/status           Update status
```

---

## 🚀 TYPICAL WORKFLOW (5 Steps)

1. **Login**: `POST /api/auth/login` → Get JWT token
2. **Create**: `POST /api/baggage-ops` → Start operation (status = InProgress)
3. **Update**: `PATCH /api/baggage-ops/{id}/count` → Repeat as bags processed
4. **Complete**: `PATCH /api/baggage-ops/{id}/complete` → End operation
   - If expected = processed → Status = Completed ✅
   - If mismatch → Status = Discrepancy ⚠️ (Alerts sent)
5. **Handle Issues**: If discrepancy, report via `/api/mishandled`

---

## ⚠️ KEY VALIDATIONS (WHAT SYSTEM PREVENTS)

```
❌ Can't create 2 inbound operations for same flight
❌ Can't process more bags than expected
❌ Can't report same bag tag twice
❌ Can't reopen closed mishandled cases
❌ Can't access operations without proper role
```

---

## 📈 STATUS VALUES - MUST MEMORIZE

### OperationStatus (3 values)
```
InProgress    → Operation ongoing
Completed     → All bags matched ✅
Discrepancy   → Mismatch detected ⚠️
```

### MishandledStatus (5 values)
```
Reported      → Just reported
Traced        → Found what happened
Recovered     → Issue resolved/bag found
Claimed       → Passenger got bag ✅
ClosedUnresolved → Couldn't resolve ❌
```

### MishandledType (4 values)
```
Lost          → Bag can't be found
Delayed       → Bag arrived late
Damaged       → Physical damage
PilferedContent → Items stolen from bag
```

### Direction (2 values)
```
Inbound       → Flight arriving (unloading)
Outbound      → Flight departing (loading)
```

---

## 💡 COMMON INTERVIEW QUESTIONS & ANSWERS

**Q: What's a discrepancy?**
A: When expected bags ≠ processed bags. System marks as "Discrepancy" and alerts supervisors.

**Q: How does your system prevent errors?**
A: Validation checks on counts, duplicate prevention, role-based access, transactional operations.

**Q: Explain the mishandled bag workflow**
A: Reported → Traced → Recovered → Claimed (or ClosedUnresolved)

**Q: What happens if RampOfficer reports a bag as Lost?**
A: Creates MishandledBaggage record, status = Reported, supervisor gets notified.

**Q: Can PassengerAgent create operations?**
A: No, only RampOfficer and Admin can create. PassengerAgent only looks up bags.

**Q: What's the difference between update count and complete?**
A: Update count = intermediate logging, Complete = end operation and check for discrepancy.

---

## 🔄 STATE TRANSITIONS (FLOWS)

### Baggage Operation Flow
```
Create (InProgress) → Update Count (multiple times) → Complete → End State:
                                                          ├─ Completed ✅ (if counts match)
                                                          └─ Discrepancy ⚠️ (if mismatch)
```

### Mishandled Baggage Flow
```
Report (Reported) → Trace (Traced) → Recover (Recovered) → Claim (Claimed) ✅
                                                        OR → ClosedUnresolved ❌
```

---

## 🎬 MOCK INTERVIEW - YOUR 2-MINUTE PITCH

*"I built the baggage handling module for FlightOps. It consists of two parts:*

*First, BaggageOperations - when a flight arrives, we create an operation to track baggage. We expect X bags and count how many we actually process. If there's a mismatch, we flag it as 'Discrepancy' and the system automatically alerts supervisors.*

*Second, MishandledBaggage - for lost, damaged, or delayed bags. When we report an issue, it goes through a workflow: Reported (just reported) → Traced (investigated) → Recovered (fixed/found) → Claimed (resolved). If we can't resolve, it goes to ClosedUnresolved.*

*I use role-based access control - RampOfficers can create and update operations, supervisors can view and investigate, passenger agents can look up bags for customer service. The system has validation to prevent errors like double-counting or duplicate reports."*

---

## 🛠️ TECHNOLOGY MENTION (If Asked)

- **Framework**: Spring Boot 4.0.6
- **Language**: Java 21
- **Database**: MySQL
- **Security**: JWT tokens + Spring Security (role-based)
- **Architecture**: Entity-Service-Controller pattern
- **ORM**: JPA/Hibernate
- **Key Features**: Transactional operations, logging (SLF4J), input validation

---

## 📱 SAMPLE REQUEST/RESPONSE (TO PRACTICE)

### Create Operation
```json
POST /api/baggage-ops
REQUEST:
{
  "flightId": "FL001",
  "direction": "Inbound",
  "totalBagsExpected": 200,
  "startTime": "2024-06-16T10:00:00"
}

RESPONSE (201):
{
  "operationId": "op-uuid-123",
  "flightNumber": "BA101",
  "status": "InProgress",
  "totalBagsExpected": 200,
  "totalBagsProcessed": 0,
  "discrepancy": 200
}
```

### Update Count
```json
PATCH /api/baggage-ops/op-uuid-123/count
REQUEST:
{
  "totalBagsProcessed": 195
}

RESPONSE (200):
{
  "operationId": "op-uuid-123",
  "totalBagsExpected": 200,
  "totalBagsProcessed": 195,
  "discrepancy": 5,
  "status": "InProgress"
}
```

### Complete (With Discrepancy)
```json
PATCH /api/baggage-ops/op-uuid-123/complete
RESPONSE (200):
{
  "status": "Discrepancy",
  "discrepancy": 5,
  "message": "5 bags unaccounted - supervisors notified"
}
```

---

## 🎯 THINGS TO EMPHASIZE IN INTERVIEW

✅ **Error Prevention** - Talk about validations  
✅ **Automation** - System notifies people automatically  
✅ **Real-time Tracking** - Multiple count updates  
✅ **Workflow Management** - States and transitions  
✅ **Role-Based Security** - Different users different access  
✅ **Problem Resolution** - Process for fixing issues  
✅ **Audit Trail** - Everything is tracked and logged  

---

## ❌ THINGS TO AVOID SAYING

❌ "I didn't understand much about the project"  
❌ "I just copied code from somewhere"  
❌ "I don't know why this works"  
❌ Too technical jargon without explaining  
❌ Making up features that don't exist  
❌ Being unsure about basic concepts  

---

## 📞 QUICK REFERENCE: API BASE CALLS

```bash
# Login
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"john.ramp@flightops.com","password":"..."}'

# All with Authorization header:
-H "Authorization: Bearer YOUR_JWT_TOKEN"

# Create operation
curl -X POST http://localhost:8080/api/baggage-ops \
  -H "Authorization: Bearer TOKEN" \
  -d '{"flightId":"FL001","direction":"Inbound","totalBagsExpected":200,"startTime":"2024-06-16T10:00"}'

# Get all
curl -X GET http://localhost:8080/api/baggage-ops \
  -H "Authorization: Bearer TOKEN"

# Update count
curl -X PATCH http://localhost:8080/api/baggage-ops/OP_ID/count \
  -H "Authorization: Bearer TOKEN" \
  -d '{"totalBagsProcessed":100}'

# Complete
curl -X PATCH http://localhost:8080/api/baggage-ops/OP_ID/complete \
  -H "Authorization: Bearer TOKEN"

# Report mishandled
curl -X POST http://localhost:8080/api/mishandled \
  -H "Authorization: Bearer TOKEN" \
  -d '{"flightId":"FL001","passengerName":"Alice","bagTagNumber":"BA-123","mishandleType":"Damaged"}'

# Update mishandled status
curl -X PATCH http://localhost:8080/api/mishandled/MH_ID/status \
  -H "Authorization: Bearer TOKEN" \
  -d '{"status":"Traced"}'
```

---

## 🎓 IF THEY ASK "ANY QUESTIONS?"

Good questions to ask:
- "What happens if we need to track baggage across multiple flights?"
- "Do we have any analytics on baggage discrepancy patterns?"
- "How do we handle peak season with higher volumes?"
- "Is there integration with other ground ops modules?"

Bad questions to ask:
- "What's Spring Boot?" (should know this!)
- "Do we use database?" (obviously!)

---

**GOOD LUCK! You've got this! 🚀**

*Last minute tip: Practice explaining your module out loud 3 times before interview. Confidence is key!*
