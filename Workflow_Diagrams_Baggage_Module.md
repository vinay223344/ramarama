# 🎬 Visual Workflow Diagrams - Baggage Module

## WORKFLOW #1: Baggage Operation - Complete Cycle

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BAGGAGE OPERATION WORKFLOW                       │
└─────────────────────────────────────────────────────────────────────┘

     FLIGHT ARRIVES
            │
            ▼
    ┌──────────────┐
    │ 1. LOGIN AS  │ Email: john.ramp@flightops.com
    │ RampOfficer  │ Role: RampOfficer
    └──────┬───────┘
           │ Get JWT Token
           ▼
    ┌──────────────────────────┐
    │ 2. CREATE OPERATION      │ POST /api/baggage-ops
    │                          │ {
    │ Status: InProgress       │   flightId: "FL001"
    │ totalBagsExpected: 200   │   direction: "Inbound"
    │ totalBagsProcessed: 0    │   totalBagsExpected: 200
    │ discrepancy: 200         │   startTime: "2024-06-16T10:00"
    └──────┬───────────────────┘ }
           │
           ▼ (Scanning bags one by one...)
    ┌──────────────────────────┐
    │ 3. UPDATE COUNT (batch1) │ PATCH /api/baggage-ops/{id}/count
    │                          │ { totalBagsProcessed: 50 }
    │ Progress: 50/200         │
    │ discrepancy: 150         │
    └──────┬───────────────────┘
           │
           ▼ (Continue scanning...)
    ┌──────────────────────────┐
    │ 4. UPDATE COUNT (batch2) │ PATCH /api/baggage-ops/{id}/count
    │                          │ { totalBagsProcessed: 100 }
    │ Progress: 100/200        │
    │ discrepancy: 100         │
    └──────┬───────────────────┘
           │
           ▼ (Final batch...)
    ┌──────────────────────────┐
    │ 5. UPDATE COUNT (batch3) │ PATCH /api/baggage-ops/{id}/count
    │                          │ { totalBagsProcessed: 195 }
    │ Progress: 195/200        │
    │ discrepancy: 5           │
    └──────┬───────────────────┘
           │
           ▼
    ┌──────────────────────────┐
    │ 6. COMPLETE OPERATION    │ PATCH /api/baggage-ops/{id}/complete
    └──────┬───────────────────┘
           │
        DECISION POINT
        │
        ├─ Expected = Processed? ─────────────┐
        │                                      │
        ▼                                      ▼
    ✅ YES                              ❌ NO (DISCREPANCY)
    Status: Completed                   Status: Discrepancy
    (Perfect match!)                    (5 bags missing!)
                                        │
                                        ▼
                                    SYSTEM ALERTS:
                                    - GroundSupervisor
                                    - RampOfficer
                                    (via notification)
```

---

## WORKFLOW #2: Mishandled Baggage - Investigation Cycle

```
┌─────────────────────────────────────────────────────────────────────┐
│                 MISHANDLED BAGGAGE WORKFLOW                         │
└─────────────────────────────────────────────────────────────────────┘

    PROBLEM DISCOVERED
    (Bag is damaged/lost/delayed/stolen)
           │
           ▼
    ┌────────────────────────────┐
    │ 1. REPORT MISHANDLED BAG   │ POST /api/mishandled
    │                            │ {
    │ Status: Reported ◯         │   flightId: "FL001"
    │ Type: Lost/Damaged/etc     │   passengerName: "Alice Johnson"
    │ Incident ID assigned       │   bagTagNumber: "BA-2024-12345"
    │                            │   mishandleType: "Damaged"
    │ Notification sent to       │ }
    │ GroundSupervisor           │
    └────────┬───────────────────┘
             │
             ▼
    ┌────────────────────────────┐
    │ 2. INVESTIGATION STARTS    │ PATCH /api/mishandled/{id}/status
    │                            │ { status: "Traced" }
    │ Status: Traced ◯→          │
    │ Supervisor tracking issue  │
    │ "Where is this bag?"       │
    └────────┬───────────────────┘
             │
             ▼
    ┌────────────────────────────┐
    │ 3. BAG FOUND/FIXED         │ PATCH /api/mishandled/{id}/status
    │                            │ { status: "Recovered" }
    │ Status: Recovered ✓        │
    │ Bag retrieved or repaired  │
    │ Ready for passenger        │
    └────────┬───────────────────┘
             │
             ▼
    ┌────────────────────────────┐
    │ 4. PASSENGER GETS BAG      │ PATCH /api/mishandled/{id}/status
    │                            │ { status: "Claimed" }
    │ Status: Claimed ✅         │
    │ CASE CLOSED SUCCESSFULLY   │
    └────────────────────────────┘

OR (If can't resolve):

    ┌────────────────────────────┐
    │ 4. CASE CANNOT BE RESOLVED │ PATCH /api/mishandled/{id}/status
    │                            │ { status: "ClosedUnresolved" }
    │ Status: ClosedUnresolved ❌│
    │ Bag not found, document it │
    │ for compensation           │
    └────────────────────────────┘

STATUS TIMELINE:
    Reported (0 hours)
        │
        ▼
    Traced (24 hours)
        │
        ▼
    Recovered (48 hours)
        │
        ▼
    Claimed (72 hours) ✅
```

---

## WORKFLOW #3: Role-Based Access Control

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ROLE-BASED ACCESS                                │
└─────────────────────────────────────────────────────────────────────┘

USER LOGS IN → JWT TOKEN ISSUED → ROLE DETERMINED → ACCESS GRANTED/DENIED

┌──────────────────────────────────────────────────────────────────────┐
│                         RAMP OFFICER                                  │
├──────────────────────────────────────────────────────────────────────┤
│ ✅ CREATE baggage operation                                           │
│ ✅ UPDATE bag count during operation                                  │
│ ✅ COMPLETE operation                                                 │
│ ✅ REPORT mishandled baggage                                          │
│ ✅ UPDATE mishandled status                                           │
│ ✅ VIEW all operations & mishandled records                           │
│                                                                       │
│ Endpoints: POST/PATCH /api/baggage-ops, POST/PATCH /api/mishandled  │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                      GROUND SUPERVISOR                                │
├──────────────────────────────────────────────────────────────────────┤
│ ✅ VIEW all baggage operations                                        │
│ ✅ VIEW all mishandled records                                        │
│ ✅ INVESTIGATE discrepancies                                          │
│ ✅ RECEIVE notifications (automatic)                                  │
│ ❌ Cannot create/update operations                                    │
│ ❌ Cannot report mishandled bags                                      │
│                                                                       │
│ Endpoints: GET /api/baggage-ops, GET /api/mishandled                │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                      PASSENGER AGENT                                  │
├──────────────────────────────────────────────────────────────────────┤
│ ✅ LOOKUP specific mishandled baggage (for customer service)         │
│ ❌ Cannot create operations                                           │
│ ❌ Cannot report mishandled bags                                      │
│ ❌ Cannot update operations                                           │
│                                                                       │
│ Endpoints: GET /api/mishandled/{bagTag}                             │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                           ADMIN                                       │
├──────────────────────────────────────────────────────────────────────┤
│ ✅ FULL ACCESS - All operations available                            │
│ ✅ Override any restrictions                                         │
│ ✅ Manage user roles                                                 │
│                                                                       │
│ Endpoints: All endpoints                                            │
└──────────────────────────────────────────────────────────────────────┘

HOW IT WORKS:
    curl -X POST /api/baggage-ops
    With Header: Authorization: Bearer <token>
    
    System checks: Is this user a RampOfficer?
    
    YES → ✅ Create operation
    NO  → ❌ Return 403 Forbidden
```

---

## WORKFLOW #4: Real-Time Baggage Count Tracking

```
┌─────────────────────────────────────────────────────────────────────┐
│              REAL-TIME BAGGAGE TRACKING DISPLAY                     │
└─────────────────────────────────────────────────────────────────────┘

FLIGHT BA101 INBOUND - UNLOADING STARTS

    Time: 10:00 AM
    ┌─────────────────────────────┐
    │ Expected: 200 bags          │
    │ Processed: 0 bags (0%)       │
    │ Remaining: 200 bags         │
    │ Discrepancy: 200 bags ⚠️    │
    └─────────────────────────────┘
    ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 0%

    Time: 10:15 AM (1st batch complete - 50 bags scanned)
    ┌─────────────────────────────┐
    │ Expected: 200 bags          │
    │ Processed: 50 bags (25%)     │
    │ Remaining: 150 bags         │
    │ Discrepancy: 150 bags ⚠️    │
    └─────────────────────────────┘
    ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 25%

    Time: 10:30 AM (2nd batch - 100 bags total)
    ┌─────────────────────────────┐
    │ Expected: 200 bags          │
    │ Processed: 100 bags (50%)    │
    │ Remaining: 100 bags         │
    │ Discrepancy: 100 bags ⚠️    │
    └─────────────────────────────┘
    ████████████████████░░░░░░░░░░░░░░░░░░░░ 50%

    Time: 10:45 AM (3rd batch - 195 bags total)
    ┌─────────────────────────────┐
    │ Expected: 200 bags          │
    │ Processed: 195 bags (97.5%)  │
    │ Remaining: 5 bags           │
    │ Discrepancy: 5 bags ⚠️      │
    └─────────────────────────────┘
    ███████████████████████████████████████░ 97.5%

    Time: 11:00 AM (COMPLETE - Problem detected!)
    ┌─────────────────────────────┐
    │ Expected: 200 bags          │
    │ Processed: 195 bags (97.5%)  │
    │ Status: DISCREPANCY ❌      │
    │ Missing: 5 bags             │
    │ Alert sent to supervisors   │
    └─────────────────────────────┘
    ███████████████████████████████████████░ 97.5%

    ALERT SENT TO:
    • Alice (GroundSupervisor)
    • John (RampOfficer)
    
    Message: "5 bags unaccounted on BA101 (Inbound)"
    Action: INVESTIGATE → Find where 5 bags are!
```

---

## WORKFLOW #5: Complete User Journey (End-to-End)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMPLETE USER JOURNEY                            │
│                   (What You Do In Interview)                        │
└─────────────────────────────────────────────────────────────────────┘

YOU (Interviewer asks): "Walk me through how you handle a damaged baggage"

YOU EXPLAIN:

┌─────────────────────────────────────────────────────────────────────┐
│ STEP 1: AUTHENTICATION                                              │
├─────────────────────────────────────────────────────────────────────┤
│ Me: "First, I'd login with my RampOfficer credentials"              │
│ Endpoint: POST /api/auth/login                                      │
│ Response: JWT token (valid for 24 hours)                            │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ STEP 2: DISCOVER DAMAGE                                             │
├─────────────────────────────────────────────────────────────────────┤
│ Me: "While unloading bags, I find one bag is ripped"                │
│ Action: Document it - passenger name, flight, bag tag               │
│ Status: Creating incident report                                    │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ STEP 3: REPORT TO SYSTEM                                            │
├─────────────────────────────────────────────────────────────────────┤
│ Me: "I report it using our API"                                     │
│ Endpoint: POST /api/mishandled                                      │
│ Payload:                                                            │
│ {                                                                   │
│   flightId: "BA101",                                                │
│   passengerName: "John Doe",                                        │
│   bagTagNumber: "BA-2024-99999",                                    │
│   mishandleType: "Damaged"                                          │
│ }                                                                   │
│ Response: Incident created, status = "Reported"                    │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ STEP 4: NOTIFICATION SENT                                           │
├─────────────────────────────────────────────────────────────────────┤
│ Me: "System automatically notifies Alice (GroundSupervisor)"        │
│ Alert: "Mishandled bag reported: Damaged"                           │
│        "Tag: BA-2024-99999 | Passenger: John Doe | Flight: BA101"  │
│ Purpose: Keep supervisor informed immediately                       │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ STEP 5: INVESTIGATION BEGINS (Supervisor)                           │
├─────────────────────────────────────────────────────────────────────┤
│ Me: "Supervisor Alice logs in and views the incident"               │
│ Endpoint: GET /api/mishandled/BA-2024-99999                         │
│ She sees: Details of damaged bag, current status = "Reported"       │
│ Action: She investigates - checks repair facility, contacts airline │
│ Updates status: PATCH /api/mishandled/{id}/status                   │
│ Payload: { status: "Traced" }                                       │
│ (Now status = "Traced" - bag location identified)                   │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ STEP 6: RESOLUTION (Repair & Recovery)                              │
├─────────────────────────────────────────────────────────────────────┤
│ Me: "Bag is repaired at maintenance facility"                       │
│ Alice updates status again:                                         │
│ Endpoint: PATCH /api/mishandled/{id}/status                         │
│ Payload: { status: "Recovered" }                                    │
│ (Now status = "Recovered" - bag is ready for passenger)             │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ STEP 7: PASSENGER CLAIMS (Final Resolution)                         │
├─────────────────────────────────────────────────────────────────────┤
│ Me: "John Doe comes to baggage claim, receives his bag"             │
│ Alice updates final status:                                         │
│ Endpoint: PATCH /api/mishandled/{id}/status                         │
│ Payload: { status: "Claimed" }                                      │
│ (Now status = "Claimed" ✅ - CASE CLOSED)                           │
│                                                                     │
│ Database Records:                                                   │
│ • Incident created & resolved: BA-2024-99999                        │
│ • Timeline tracked: Reported → Traced → Recovered → Claimed        │
│ • Audit trail complete: When & who updated each time                │
└─────────────────────────────────────────────────────────────────────┘

SUMMARY:
✅ Discovered problem
✅ Reported to system  
✅ System alerted supervisor
✅ Supervisor investigated
✅ Issue resolved
✅ Passenger satisfied
✅ Complete audit trail maintained
```

---

## WORKFLOW #6: Discrepancy Detection & Resolution

```
┌─────────────────────────────────────────────────────────────────────┐
│              DISCREPANCY DETECTION WORKFLOW                         │
└─────────────────────────────────────────────────────────────────────┘

SCENARIO: BA101 unloading - expected 200 bags, only found 195

    OPERATION IN PROGRESS (100 bags so far)
    ┌─────────────────────────┐
    │ Expected: 200           │
    │ Processed: 100          │
    │ Status: InProgress      │
    └─────────────────────────┘
    
    ... scanning continues ...
    
    ┌─────────────────────────┐
    │ Expected: 200           │
    │ Processed: 195          │
    │ Status: InProgress      │
    └─────────────────────────┘
    
    RAMP OFFICER: "Complete this operation"
    Call: PATCH /api/baggage-ops/op-123/complete
    
    └──── SYSTEM LOGIC ──────────────────────────────┐
         |                                            │
         ├─ Calculate discrepancy:                   │
         │  200 - 195 = 5 bags missing               │
         │                                            │
         ├─ Decision:                                │
         │  if (discrepancy == 0)                    │
         │    status = "Completed" ✅                │
         │  else                                     │
         │    status = "Discrepancy" ⚠️              │
         │                                            │
         ├─ Send Alerts:                             │
         │  to: GroundSupervisor, RampOfficer        │
         │  message: "5 bags unaccounted - BA101"    │
         │                                            │
         └────────────────────────────────────────────┘
    
    SYSTEM RESPONSE:
    ┌─────────────────────────────────┐
    │ Status: DISCREPANCY             │
    │ Expected: 200 bags              │
    │ Processed: 195 bags             │
    │ Missing: 5 bags ⚠️              │
    │ Alerts sent: 3 supervisors      │
    │ Investigation required: YES     │
    └─────────────────────────────────┘
    
    SUPERVISOR INVESTIGATION:
    1. Check if 5 bags found elsewhere → Report as mishandled
    2. Search cargo area → Maybe delayed on another flight
    3. Review process → Maybe counting error
    
    RESOLUTION OPTIONS:
    
    Option A: All 5 bags found
    ────────────────────────
    Report each as MishandledBaggage:
    - Type: "Delayed" (found later)
    - Update status through workflow
    - Case closed after resolution
    
    Option B: 3 bags found, 2 permanently lost
    ──────────────────────────────────────
    Report found bags as "Delayed"
    Report lost bags as "Lost" → ClosedUnresolved
    Complete investigation
    
    Option C: Counting error (actually all 200 there)
    ──────────────────────────────────
    Investigation reveals mistake in count
    Manually update: totalBagsProcessed = 200
    Status changes to: Completed ✅
    
    FINAL STATE:
    All 5 bags accounted for or documented
    Status: Completed or Discrepancy (documented)
    Audit trail: Complete investigation history
```

---

## WORKFLOW #7: Data Validation & Error Prevention

```
┌─────────────────────────────────────────────────────────────────────┐
│                   ERROR PREVENTION WORKFLOW                         │
└─────────────────────────────────────────────────────────────────────┘

VALIDATION 1: Duplicate Operation Prevention
──────────────────────────────────────────

REQUEST:
  curl -X POST /api/baggage-ops
  { flightId: "BA101", direction: "Inbound", ... }

CHECK:
  ├─ Does operation with (flightId="BA101" AND direction="Inbound") exist?
  │
  ├─ YES → REJECT ❌
  │  Response: 409 Conflict
  │  Message: "A Inbound baggage operation already exists for flight BA101"
  │
  └─ NO → CREATE ✅
     Response: 201 Created
     New operation created

RESULT: Never have 2 inbound operations for same flight


VALIDATION 2: Processed > Expected Prevention
──────────────────────────────────────────

REQUEST:
  curl -X PATCH /api/baggage-ops/op-123/count
  { totalBagsProcessed: 250 }

CHECK:
  ├─ Is processed (250) ≤ expected (200)?
  │
  ├─ NO → REJECT ❌
  │  Response: 400 Bad Request
  │  Message: "Bags processed (250) cannot exceed bags expected (200)"
  │
  └─ YES → UPDATE ✅
     Response: 200 OK
     Count updated

RESULT: Never log more bags than expected


VALIDATION 3: Duplicate Bag Tag Prevention
──────────────────────────────────────────

REQUEST:
  curl -X POST /api/mishandled
  { bagTagNumber: "BA-2024-99999", ... }

CHECK:
  ├─ Does mishandled record with this bagTagNumber exist?
  │
  ├─ YES → REJECT ❌
  │  Response: 409 Conflict
  │  Message: "Bag tag BA-2024-99999 is already reported"
  │
  └─ NO → CREATE ✅
     Response: 201 Created
     New incident created

RESULT: Never have 2 reports for same bag


VALIDATION 4: Status Transition Prevention
───────────────────────────────────────────

REQUEST:
  curl -X PATCH /api/mishandled/mh-123/status
  { status: "Traced" }

CURRENT STATUS CHECK:
  ├─ Is current status "Claimed"?
  │  ├─ YES → Cannot update (case is closed)
  │  │  Response: 400 Bad Request
  │  │  Message: "Cannot update a closed/claimed baggage case"
  │  │
  │  └─ NO → Can update ✅
  │
  └─ Is current status "ClosedUnresolved"?
     ├─ YES → Cannot update (case is closed)
     │  Response: 400 Bad Request
     │
     └─ NO → Can update ✅

RESULT: Prevents reopening closed cases


SUMMARY OF PROTECTIONS:
✅ No duplicate operations per flight+direction
✅ No more bags logged than expected
✅ No duplicate bag tag reports
✅ No invalid status transitions
✅ Role-based access prevents unauthorized changes
✅ Transactional operations ensure data consistency
```

---

## QUICK API FLOW DIAGRAM

```
USER LOGIN
    ↓ POST /api/auth/login
RECEIVE JWT TOKEN
    ↓ Use token in all requests
BAGGAGE OPERATION FLOW
    ├→ POST /api/baggage-ops (Create)
    ├→ PATCH /api/baggage-ops/{id}/count (Update)
    ├→ PATCH /api/baggage-ops/{id}/count (Update again)
    ├→ PATCH /api/baggage-ops/{id}/complete (End)
    │   ├─ If OK: Status = Completed ✅
    │   └─ If issue: Status = Discrepancy ⚠️ (Alert sent)
    └→ GET /api/baggage-ops (View all)

IF DISCREPANCY OR PROBLEM:
    ├→ POST /api/mishandled (Report)
    ├→ GET /api/mishandled (View incidents)
    ├→ GET /api/mishandled/{bagTag} (Look up one)
    └→ PATCH /api/mishandled/{id}/status (Update: Reported→Traced→Recovered→Claimed)
```

---

**These diagrams help you visualize the workflows during your interview! 🎯**
