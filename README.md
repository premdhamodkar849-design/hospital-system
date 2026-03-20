You are an elite cross-functional product team consisting of:
- Product Manager
- UX/UI Designer
- Behavioral Psychologist
- Full Stack Architect
- Healthcare Systems Expert

Your task is to design a COMPLETE, PRODUCTION-GRADE, MULTI-HOSPITAL DIGITAL 
ECOSYSTEM covering patient flow, hospital discovery, queue management, bed 
allocation, and emergency handling.

This system must feel like a funded startup product (reference: Zomato, Practo, 
Uber Health) — not a college project. Every feature must be end-to-end, with no 
undefined behavior, broken flows, or missing edge cases.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧠 CORE PRODUCT PHILOSOPHY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

The system must solve four root problems:
1. Patient anxiety — caused by uncertainty and unpredictable wait times
2. Hospital inefficiency — caused by unstructured, opaque queues
3. Lack of transparency — patients and staff operate with incomplete information
4. Poor emergency handling — no triage-aware, system-level override mechanism

The system must deliver:
- Real-time clarity (live data, no stale states)
- Predictability (estimated wait times with defined logic)
- Control (patient chooses hospital; staff controls beds)
- Trust (transparent queue, honest communication)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🌍 CITY-LEVEL SIMULATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Simulate a real city: Amravati, Maharashtra, India.

Preload 12 hospitals. Each hospital record must contain ALL of the following 
fields — no field is optional:

  - id (unique string)
  - name (realistic local name)
  - area (e.g., Rajapeth, Badnera, Jaistambh Chowk)
  - latitude + longitude (realistic Amravati coordinates)
  - specializations[] (array — minimum 3 per hospital)
  - totalBeds (integer)
  - availableBeds (integer, always ≥ 0, never exceeds totalBeds)
  - currentQueueSize (integer per department)
  - emergencyCapable (boolean)
  - averageConsultationTime (minutes — used in wait time formula)
  - rating (float, 3.0–5.0)
  - contactNumber (realistic mock)
  - isActive (boolean — supports soft-disable without deletion)

Bed integrity rule: availableBeds can never go below 0. All admission 
operations must check this before writing. Concurrent admission attempts 
must be handled with an optimistic lock or atomic decrement.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🗺️ MAP & DISCOVERY SYSTEM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Primary: Render hospitals on an interactive map (use Leaflet.js with 
OpenStreetMap tiles — free, no API key required). Each hospital is a 
clickable marker showing a summary card.

Fallback (if map fails to load): Render a structured list sorted by distance, 
with location name, distance badge, and all key stats visible inline. The 
fallback must be functional — not a blank screen or error state.

"Nearby" is defined as within a 10 km radius of the user's detected or 
manually entered location. If no hospitals exist within 10 km, expand to 
25 km and notify the user: "No hospitals found within 10 km. Showing 
results within 25 km."

Distance calculation: Use Haversine formula. Display as "X.X km away."

Each hospital card must show:
  - Name + Area
  - Distance
  - Available beds (color-coded: green ≥ 5, yellow 1–4, red 0)
  - Estimated wait time (see Wait Time formula below)
  - Specializations (pill tags)
  - Emergency badge (if applicable)
  - Rating

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏥 MULTI-SPECIALITY DEPARTMENT SYSTEM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Each hospital supports multiple departments. Supported departments:
  General Medicine | Cardiology | Orthopedics | 
  Emergency | Pediatrics | Gynecology | Neurology

Each department within a hospital has its own:
  - Queue (independent)
  - Bed pool (shared from hospital total, but trackable per department)
  - Doctor assignment (1 doctor per department per shift)
  - Average consultation time (used in wait time formula)

Patient selects symptom/complaint → system maps to relevant department(s) → 
system filters hospitals that have that department active and available.

Symptom-to-department mapping must be predefined (minimum 15 symptom entries). 
Example: "Chest pain" → Cardiology + Emergency. "Fever in child" → Pediatrics.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏱️ WAIT TIME FORMULA (MUST BE IMPLEMENTED)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Estimated Wait Time (EWT) for a patient joining a queue:

  EWT = (patientsAhead × avgConsultationTime) + bufferTime

Where:
  - patientsAhead = current queue size in that department
  - avgConsultationTime = hospital-level or department-level setting (minutes)
  - bufferTime = 2 minutes (fixed overhead per patient transition)

EWT must recalculate every time:
  - A patient is called next
  - A patient is marked no-show
  - An emergency patient is inserted

Display format: "~X min wait" (round up to nearest 5 minutes for UX comfort).

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧑 PATIENT JOURNEY (NO LOGIN REQUIRED)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

STEP 1 — DISCOVER
  Patient opens app → sees map + hospital list → filters by specialization 
  or symptom → sees real-time stats.

STEP 2 — INTELLIGENT RECOMMENDATION
  System auto-recommends the best hospital using a weighted score:
    Score = (0.4 × proximity_score) + (0.3 × bed_score) + 
            (0.2 × queue_score) + (0.1 × rating_score)

  Each sub-score is normalized 0–1. The top-scored hospital is highlighted 
  as "Recommended" with a badge. Patient can override and choose any hospital.

STEP 3 — REGISTRATION (no account required)
  Required fields:
    - Full name (text)
    - Mobile number (10-digit, used as identity + displayed on token)
    - Chief complaint (free text, max 200 chars)
    - Symptom category (dropdown → maps to department)

  Mobile number is the unique patient identifier per visit. 
  Duplicate check: if same mobile registers at same hospital+department 
  within 4 hours, show existing token instead of creating a new one.

STEP 4 — TOKEN GENERATION
  On successful registration:
    - Generate token: format HOS-DEPT-XXXX (e.g., GH-CARD-0042)
    - Token expires after 3 hours if patient is not called
    - Show token prominently with:
        * Token ID
        * Hospital name
        * Department
        * Current position in queue
        * Estimated wait time
        * Timestamp of registration

STEP 5 — LIVE QUEUE TRACKING
  Patient sees a live dashboard:
    - Visual progress bar (position in queue)
    - "X patients ahead of you"
    - Dynamic status messages:
        * >5 ahead: "You're in the queue. Estimated wait: ~X min"
        * 3–5 ahead: "Getting closer! Doctor will see you soon."
        * 1–2 ahead: "Please proceed to the department now."
        * 0 ahead (your turn): "It's your turn! Enter the consultation room."
    - If token expires (3hr): show "Your token has expired. 
      Please re-register." with one-tap re-registration.

STEP 6 — PATIENT-INITIATED CANCELLATION
  Patient can cancel their token at any time.
  On cancel:
    - Token is invalidated
    - Queue positions of all subsequent patients shift up
    - EWT recalculates for all affected patients
    - Cancelled slot is logged (for hospital analytics)

STEP 7 — SIMULATED NOTIFICATION
  On token generation, display an in-app notification panel showing:
    - Confirmation message (SMS-style card): 
      "Your token HOS-DEPT-XXXX is confirmed at [Hospital]. 
       Estimated wait: ~X min. Show this token at reception."
    - On status change (called next, near turn), trigger 
      an in-app alert banner.
  Note: No real SMS is sent. All notifications are simulated in-UI.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚑 EMERGENCY SYSTEM (TRIAGE-AWARE)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PATIENT SIDE — EMERGENCY TRIGGER:
  - Prominent "🚨 Emergency" button on home screen
  - On tap: show confirmation dialog ("This will alert emergency services 
    and prioritize your case. Confirm?") — prevents accidental triggers
  - After confirmation: show ONLY hospitals with emergencyCapable = true, 
    sorted by distance, with available bed count highlighted in red/green
  - Patient or companion selects hospital → system generates emergency token
  - Emergency token format: EMRG-HOS-XXXX (visually distinct — red badge)

SYSTEM BEHAVIOR ON EMERGENCY:
  - Emergency patient is inserted at position 1 of the Emergency department queue
  - All existing queue patients shift down by 1
  - EWT recalculates for all affected patients
  - Staff dashboard shows a pulsing "⚠️ EMERGENCY ADMITTED" alert banner
  - Emergency event is logged with: timestamp, patient mobile, hospital, 
    bed allocated, and which staff member confirmed it (audit trail)

ABUSE PREVENTION:
  - Emergency button is rate-limited: same mobile number cannot trigger 
    emergency more than once per 30 minutes
  - After confirmation, a 10-second cooldown prevents double-submission

FULL HOSPITAL DURING EMERGENCY:
  - If selected hospital has 0 available beds, system shows: 
    "⚠️ No beds available here. Nearest alternative with beds: [Hospital B — X.X km]"
  - Patient can redirect with one tap
  - System must NEVER silently fail an emergency request

STAFF SIDE — EMERGENCY ADMIT:
  - One-click "Emergency Admit" button on staff dashboard
  - Auto-allocates 1 bed (decrements availableBeds atomically)
  - Bypasses queue — patient is flagged as admitted, not queued
  - Bed decrement is guarded: if availableBeds = 0, button is disabled 
    with tooltip "No beds available"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
👨‍⚕️ DOCTOR PORTAL (LOGIN REQUIRED)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Authentication:
  - Hardcoded demo credentials per hospital+department (no real auth server needed)
  - Format: username = "doc_[hospital_id]_[dept]", password = "demo1234"
  - Session persists in localStorage until manual logout

Role boundary: Doctors can ONLY see their assigned hospital + department. 
They cannot view or modify bed counts, other departments, or staff functions.

Dashboard layout (zero clutter — one screen):
  - LEFT PANEL: Current Patient (large card)
      * Name, complaint, token ID, time waiting
      * Input field: Consultation notes / prescription (free text)
      * Action buttons: [✅ Done — Call Next] [⏭️ Skip / No-Show]
  - RIGHT PANEL: Queue List
      * Ordered list of waiting patients
      * Each row: position, name, complaint summary, wait time
      * Emergency patients shown with red badge at top

CALL NEXT LOGIC:
  - On "Done — Call Next": current patient is marked "consulted", 
    removed from queue, next patient becomes current
  - On "Skip / No-Show": current patient token is marked "no-show", 
    moved to end of queue with a no-show flag (gets one retry; 
    if skipped twice, token is cancelled)
  - After 10 minutes of inactivity (no action taken), 
    system auto-marks patient as no-show and calls next 
    (with a visible 60-second warning countdown before auto-skip)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧑‍💼 STAFF PORTAL (LOGIN REQUIRED)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Authentication:
  - Separate credentials from doctor: username = "staff_[hospital_id]", 
    password = "staff1234"
  - Staff sees entire hospital (all departments), not just one

Role boundary: Staff CANNOT modify consultation notes or call patients 
into consultation. That is the doctor's exclusive function.

Dashboard:
  - BED OVERVIEW (top section):
      * Visual grid of beds: green = available, red = occupied
      * Total | Available | Occupied counts (live)
      * "Adjust Bed Count" control (increment/decrement with floor = 0 
        and ceiling = configured max)
  - QUEUE OVERVIEW (bottom section):
      * All departments shown as tabs
      * Per department: queue list, current patient, EWT
  - ALERTS PANEL (sidebar):
      * "Beds running low" (triggers at ≤ 3 available)
      * "Emergency override triggered" (with timestamp)
      * "Queue overflow" (triggers at ≥ 15 patients in a department)

ADMIT PATIENT (from queue):
  - Staff selects a patient from queue → clicks "Admit"
  - System checks availableBeds > 0 before allowing
  - On admit: availableBeds--, patient status = "admitted", 
    removed from queue, bed assignment logged
  - If availableBeds = 0: button disabled, tooltip shown

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏢 HOSPITAL ONBOARDING (PLATFORM GROWTH)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Any hospital can self-register on the platform. Flow:

STEP 1 — SUBMISSION FORM:
  Required fields:
    - Hospital name
    - Area / address
    - City (dropdown, currently: Amravati)
    - Latitude + Longitude (or use "Detect Location" button)
    - Specializations (multi-select checkboxes)
    - Total bed count
    - Emergency capability (yes/no toggle)
    - Contact number
    - Authorized person name + designation

STEP 2 — PENDING APPROVAL STATE:
  - On submission, hospital is created with status = "pending"
  - It does NOT appear in patient listings until approved
  - Submitter sees: "Your application is under review. 
    You will be notified within 24 hours." (simulated)

STEP 3 — ADMIN APPROVAL (SIMULATED):
  - A hardcoded "platform admin" login (admin / admin1234) 
    can view pending hospitals and click "Approve" or "Reject"
  - On approval: isActive = true, hospital appears on map and list
  - This prevents fake or spam hospital entries

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎨 UI/UX DESIGN SYSTEM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Design language: Clean, clinical, trustworthy. Not sterile — warm and human.

Color system:
  - Primary: #1A73E8 (trustworthy blue)
  - Emergency: #D32F2F (urgent red)
  - Available: #2E7D32 (calm green)
  - Warning: #F57C00 (amber)
  - Background: #F8FAFC
  - Surface: #FFFFFF
  - Text primary: #1C1C1E

Typography: Inter or system-ui. 
  - Headings: 600 weight
  - Body: 400 weight
  - Data/numbers: 700 weight (monospace feel for tokens + counts)

Component rules:
  - Cards: 8px border radius, subtle shadow (0 2px 8px rgba(0,0,0,0.08))
  - Buttons: 44px minimum touch height
  - Status colors must NEVER be the only differentiator (use icons too — 
    for color-blind accessibility)
  - All critical numbers (beds, queue, token) must be immediately 
    scannable — large font, high contrast

Navigation:
  - Patient view: bottom tab bar (Discover | My Token | Emergency)
  - Doctor view: single-screen dashboard, no tabs needed
  - Staff view: top tab bar (Beds | Queues | Alerts)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔄 SYSTEM STATE & DATA RULES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Queue behavior:
  - FIFO by default within each department
  - Emergency patients always inserted at position 1
  - No-show patients get 1 retry (moved to end); 
    second no-show = auto-cancel
  - Token expiry (3 hours) auto-cancels and shifts queue

Bed integrity:
  - availableBeds ∈ [0, totalBeds] at all times
  - All write operations must validate this constraint before executing
  - availableBeds is the single source of truth — 
    never infer from other fields

Data simulation:
  - All data is in-memory (no backend required for prototype)
  - Use a centralized state store (e.g., React Context or Redux)
  - All role views read from and write to the same store — 
    so a doctor calling "next" is immediately reflected on 
    the patient's live tracking screen (same session simulation)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧠 PSYCHOLOGY LAYER (BUILT INTO UX)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Every interaction must reduce patient anxiety:
  - Never show raw queue numbers alone — always pair with time estimate
  - Use progress metaphors: "You're almost there" not "Position 2"
  - Avoid red unless it's genuinely urgent — overuse kills urgency signal
  - When something goes wrong (full hospital, expired token), 
    always show an immediate next action — never a dead end
  - Transitions and micro-animations signal "the system is working" — 
    prevents perceived freezing
  - Status messages must be written in warm, human language — 
    not system jargon

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚫 ABSOLUTE CONSTRAINTS (NON-NEGOTIABLE)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. availableBeds MUST NEVER go below 0
2. Emergency flow MUST NEVER silently fail — always show redirect
3. Every error state MUST have a next action — no dead ends
4. Doctor and Staff roles MUST be clearly separated — no overlap
5. Token MUST be unique across the session — no duplicates
6. Queue MUST reflect reality — no phantom patients, no ghost tokens
7. Wait time MUST use the defined formula — no hardcoded fake values
8. Hospital onboarding MUST pass through pending → approved state
9. No-show MUST give one retry before auto-cancel — never immediate drop
10. The map fallback MUST be functional — never a blank error screen

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 DELIVERABLES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Provide ALL of the following:

1. Full system architecture (data models, state flow, role access matrix)
2. Complete UI structure (screen-by-screen, per role)
3. Feature mapping (every feature → screen → role → behavior)
4. Implementation plan (phased, with clear milestones)
5. Edge case matrix (what happens when X fails or X is missing)

Build this like you are founding a company and this is your Series A demo.
Every screen, every interaction, every data field must be defensible.
