Include in this file the 7 steps for Iteration 2

This iteration focuses on one part of the AIDAP system from Iteration 1: the Core Interaction Subsystem (Conversation & Notifications). This subsystem realizes both UC-1 and UC-2 from Iteration 1 and refines the design with more detailed components and responsibilities.

# Step 1
## Use Cases (for this element)

    UC-1 – Query Academic Information (inside the subsystem)
        - Primary actor: Student (via Web/Mobile/Voice client)
        - Goal: Ask a question such as “When is my next exam?” and receive a correct, contextual answer within ~2 seconds.
        - Key steps (from the subsystem’s point of view):
            1. API Gateway forwards an authenticated query (userId, message) to the Core Interaction Subsystem.
            2. The subsystem loads user context (enrolments, preferences) from profile/history data.
            3. AI/NLP interprets the natural-language message and extracts intent/entities.
            4. The subsystem calls Integration Layer adapters (LMS/Registration/Calendar) for the required academic data.
            5. AI/NLP generates a natural-language answer using the retrieved data and context
            6. The subsystem logs the interaction and returns the answer back to the API Gateway.
        - Related requirements: R1, R2, R3, R5, R6, RS1, RS5, RS6, RS7, RS8, RS10, RS13

    UC-2 – Receive Notifications and Alerts (inside the subsystem)
        - Primary actors:
            - Student (passive receiver of notifications)
            - External Data Systems (LMS, Registration, Calendar) that generate relevant events
        - Goal: Notify students about upcoming deadlines, schedule changes, and announcements via chat, mobile, or email.
        - Key steps (from the subsystem’s point of view):
            1. The subsystem receives events from the Integration Layer (e.g., new assignment, due date approaching, schedule change).
            2. Events are normalized, enriched with context, and queued for processing.
            3. The subsystem determines which students should be notified and which channels to use (chat, push, email).
            4. Notifications are dispatched via the connected channels and logged for analytics.
        Related requirements: R1, R2, R3, RS2, RS5, RS6, RS9, RS10

## Quality Attributes (focused on this element)
    
| ID   | Quality Attribute              | Description                                                                                                                                                |
|------|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| QA-1 | Performance & Scalability      | The subsystem must answer most UC-1 queries and process UC-2 notification triggers in ≤ 2 seconds on average, and scale horizontally to support thousands of concurrent users. |
| QA-2 | Reliability & Fault Tolerance  | The subsystem must continue to operate (possibly with degraded answers) when LMS/Registration/Calendar/Email services are slow or temporarily unavailable, using retries and queues. |
| QA-3 | Security & Privacy             | The subsystem must respect authenticated identities and roles, avoid leaking other users’ data in responses/notifications, and log only what is allowed by privacy policies. |
| QA-4 | Personalization                | Responses and notifications should leverage profile and history to be context-aware (relevant courses, preferred channels, timing).                        |

## Constraints (relevant to this element)
| ID   | Constraint Title                        | Description                                                                                                                                    |
|------|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| CON-1| Access via Integration Layer            | All academic data must be accessed through the Integration Layer adapters (LMS/Registration/Calendar/Email), not directly from the subsystem. |
| CON-2| Replaceable AI/NLP Providers            | AI/NLP providers must be replaceable/configurable (model versions and keys), without changing client code.                                    |
| CON-3| Cloud-Native Deployment Requirement     | The subsystem must remain cloud-native and deployable as part of the AIDAP backend service cluster.                                           |

---

# Step 2  
## Element Name and Responsibilities (Iteration 2 Focus)

Element name: **Core Interaction Subsystem (Conversation & Notifications)**

Responsibilities (refined for this iteration):

- Handle all **conversational queries** related to academic information (UC-1) once the API Gateway has authenticated and authorized the user.
- Use **AI/NLP** to interpret natural-language messages, extract intent/entities, and generate natural-language responses.
- Access **user context** (enrolments, preferences, history) for both queries and notifications to support personalization.
- Coordinate **data retrieval** from LMS, Registration, and Calendar systems via the Integration Layer.
- Manage **notifications and alerts** (UC-2) triggered by external events, including scheduling, fan-out to users, and multi-channel delivery (chat, mobile, email).
- Record **interaction and notification history** for analytics and future personalization.

This subsystem is the main realization of **UC-1 – Query Academic Information** and **UC-2 – Receive Notifications and Alerts** for AIDAP.

---

# Step 3  
## Reference Architecture / Patterns for This Element

The Core Interaction Subsystem continues to follow the **layered, service-oriented architecture** defined in Iteration 1, but is refined internally using the following patterns and styles:

- **Controller pattern (for UC-1)**  
  The **Conversation Service** acts as a controller for student queries, orchestrating calls to context, AI/NLP, and Integration Layer adapters.

- **Event-driven / message-queue style (for UC-2)**  
  **Event Intake & Scheduler** and **Notification Service** implement an asynchronous, event-driven flow for notifications and alerts.

- **Strategy pattern (for notification channels)**  
  A **NotificationChannel** interface with concrete strategies (e.g., ChatChannel, PushChannel, EmailChannel) is used to support multiple delivery channels and future extension.

- **Facade-style context component**  
  The **Context & Personalization Service** acts as a facade over the User Profile Store and Conversation History Store, providing higher-level operations such as:
  - `getUserContext(userId)`
  - `getNotificationPreferences(userId)`
  - `recordInteraction(userId, request, response)`
  - `recordNotification(userId, notificationData)`

- **Encapsulation & restricted dependencies**  
  Subcomponents only depend on:
  - Context & Personalization Service  
  - AI / NLP Adapter  
  - Integration Layer adapters (LMS, Registration, Calendar, Email)  

  They do **not** directly access external systems or raw data stores, which supports reliability, maintainability, and clear boundaries.

---

# Step 4  
## Decomposition of the Core Interaction Subsystem

### 4.1 Subcomponents and Responsibilities

| Subcomponent                        | Description / Responsibilities                                                                                                                                         | Layer (logical)          | Related Use Cases |
|------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|-------------------|
| Conversation Service               | Main controller for UC-1. Receives `(userId, message)` from the API Gateway, retrieves user context, calls AI / NLP Adapter for intent and answer, queries Integration Layer adapters, and returns final answer. | Application Services     | UC-1              |
| Notification Service               | Main controller for UC-2. Consumes scheduled notification jobs, determines target users and channels, and dispatches notifications via NotificationChannel strategies.                                        | Application Services     | UC-2              |
| Event Intake & Scheduler           | Listens for events from Integration Layer (LMS/Registration/Calendar). Normalizes events, calculates when notifications should be sent, and queues jobs for Notification Service.                            | Application Services     | UC-2              |
| Context & Personalization Service  | Provides high-level APIs to load user profiles, enrolments, preferences, and history; records interactions/notifications; centralizes personalization rules and privacy enforcement.                         | App Services (with access to Data & Infrastructure) | UC-1, UC-2        |
| AI / NLP Adapter                   | Encapsulates calls to external AI/NLP providers. Offers `interpret(message, context)` and `generateAnswer(data, context)` and handles configuration (model versions, keys), timeouts, and errors.          | Application Services     | UC-1, UC-2 (optional for templated notifications) |
| NotificationChannel strategies     | Interface and concrete channel implementations (ChatChannel, PushChannel, EmailChannel). Used by Notification Service to send messages through different delivery channels.                                | Application Services     | UC-2              |

### 4.2 External Dependencies (from Iteration 1)

The subsystem depends on, but does not own:

- **Integration Layer Adapters:** LMS Adapter, Registration Adapter, Calendar Adapter, Email Adapter.  
- **Data & Infrastructure Layer:** User Profile Store, Conversation History Store, Config & Model Registry.

These are reused from Iteration 1 and accessed only through well-defined interfaces.

---

# Step 5  
## Component & Connector View (Core Interaction Subsystem)

At this iteration, the Application Services layer from Iteration 1 is refined by expanding the Core Interaction Subsystem into its internal subcomponents and showing their connections to other layers:

- **Presentation / Channels → AIDAP API Gateway → Core Interaction Subsystem**
  - Web Chat Client, Mobile App Client, and Voice Assistant Connector call the API Gateway.
  - API Gateway forwards UC-1 queries to **Conversation Service** and UC-2–related triggers (where applicable) to **Notification Service**.

- **Core Interaction Subsystem → Context & Personalization → Data & Infrastructure**
  - Conversation Service and Notification Service call Context & Personalization Service.
  - Context & Personalization Service reads/writes from:
    - User Profile Store  
    - Conversation History Store  

- **Core Interaction Subsystem → AI / NLP Adapter → AI Provider(s)**
  - Conversation Service calls AI / NLP Adapter for:
    - `interpret(message, context)`  
    - `generateAnswer(data, context)`

- **Core Interaction Subsystem → Integration Layer Adapters → External Systems**
  - Conversation Service calls Registration/LMS/Calendar Adapters to fetch academic data.
  - Event Intake & Scheduler receives events from LMS/Registration/Calendar Adapters.
  - Notification Service uses Email Adapter (and chat/push channels) to send alerts.


![Core Interaction Subsystem – Iteration 2 C&C View](<images/Layers_picture_from_images.png>)

### 5.1 Subsystem Elements Summary

| Element / Component                | Key Connectors / Interactions                                                                                           |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| Conversation Service              | Receives queries from API Gateway; calls Context & Personalization, AI / NLP Adapter, and LMS/Registration/Calendar Adapters. |
| Notification Service              | Receives jobs from Event Intake & Scheduler; calls Context & Personalization, NotificationChannel strategies, Email Adapter.   |
| Event Intake & Scheduler          | Receives events from LMS/Registration/Calendar Adapters; enqueues notification jobs for Notification Service.           |
| Context & Personalization Service | Communicates with User Profile Store and Conversation History Store; used by Conversation and Notification Services.    |
| AI / NLP Adapter                  | Called by Conversation Service (and optionally Notification Service) to interpret queries and generate text responses.  |
| NotificationChannel strategies    | Implement channel-specific sending (chat, push, email); used by Notification Service.                                   |

---

# Step 6  
## Behaviour for Main Scenario (UC-1 and UC-2)

### 6.1 UC-1 – Query Academic Information (Core Interaction Subsystem)

**Scenario:** Student asks “When is my next exam for CPS101?”

High-level steps within this subsystem:

1. **API Gateway → Conversation Service**  
   - API Gateway forwards an authenticated query: `handleQuery(userId, text, channel)`.

2. **Conversation Service → Context & Personalization Service**  
   - Calls `getUserContext(userId)` to retrieve profile, enrolments (including CPS101), preferences, and recent history.

3. **Conversation Service → AI / NLP Adapter (interpretation)**  
   - Calls `interpret(text, userContext)` to identify intent (e.g., `GetNextExam`) and entities (`course=CPS101`).

4. **Conversation Service → Integration Layer (Registration Adapter)**  
   - Calls `getExamSchedule(userId, "CPS101")` via Registration Adapter; receives upcoming exam date(s) for CPS101.

5. **Conversation Service → Integration Layer (LMS Adapter)**  
   - Optionally calls `getCourseDetails("CPS101")` for additional context (course title, section, etc.).

6. **Conversation Service → AI / NLP Adapter (answer generation)**  
   - Calls `generateAnswer(examInfo, courseDetails, userContext)` to build a natural-language response.

7. **Conversation Service → Context & Personalization Service (logging)**  
   - Calls `recordInteraction(userId, text, answerText)` to append to Conversation History Store.

8. **Conversation Service → API Gateway**  
   - Returns `answerText` to the API Gateway, which forwards it to the Web/Mobile/Voice client for display to the student.


![UC-1 Sequence – Core Interaction Subsystem, Iteration 2](<images/Sequence_diagram.png>)

---

### 6.2 UC-2 – Receive Notifications and Alerts (Core Interaction Subsystem)

**Scenario:** Deadline reminder for an upcoming assignment in CPS101.

1. **Integration Layer → Event Intake & Scheduler**  
   - LMS Adapter detects or receives an event: “Assignment A1 for CPS101 due in 24 hours” and forwards it to Event Intake & Scheduler.

2. **Event Intake & Scheduler (normalize and schedule)**  
   - Normalizes the event, determines when the reminder should be sent, and enqueues a notification job for Notification Service.

3. **Event Intake & Scheduler → Notification Service**  
   - At the scheduled time, passes a job containing course, affected students, and deadline info.

4. **Notification Service → Context & Personalization Service**  
   - Requests enrolment lists (who is in CPS101) and notification preferences (preferred channels, quiet hours) for each student.

5. **Notification Service → NotificationChannel strategies**  
   - For each student, uses the selected channel implementations (ChatChannel, PushChannel, EmailChannel) to send the reminder.

6. **Notification Service → Context & Personalization Service (logging)**  
   - Records which notifications were sent (or failed) for analytics and future personalization (e.g., “this user ignores email but responds to chat”).

---

# Step 7  
## Design Decisions and Driver Coverage (Iteration 2)

### 7.1 Design Decisions – Iteration 2

| ID    | Design Decision                                                                                     | Motivating Drivers (UCs / QAs / Constraints)                                                              | Consequences (Impact)                                                                                                           |
|-------|------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| D2-1  | Split the original Conversation Orchestrator into a focused **Conversation Service** plus shared Context & Personalization and AI Adapter. | UC-1, QA-1 Performance, QA-4 Personalization, CON-1                                                       | Clarifies responsibilities, keeps UC-1 path lean, and allows shared personalization logic to be reused by other services.      |
| D2-2  | Introduce **Event Intake & Scheduler** as a separate component from Notification Service.            | UC-2, QA-1 Performance & Scalability, QA-2 Reliability                                                   | Offloads scheduling and retries from Notification Service; improves reliability and makes notification flow more scalable.     |
| D2-3  | Implement **NotificationChannel** strategies (Chat, Push, Email) behind a common interface.          | UC-2, QA-4 Personalization, CON-3 Cloud-Native Deployment                                                | Enables multiple channels with minimal coupling; new channels can be added without changing core notification logic.           |
| D2-4  | Centralize user/profile/history access in **Context & Personalization Service**.                     | UC-1, UC-2, QA-3 Security & Privacy, QA-4 Personalization                                                | Enforces privacy rules and consistent personalization across conversations and notifications; simplifies data access patterns. |
| D2-5  | Keep **AI / NLP Adapter** as a dedicated component with simple operations (interpret, generateAnswer).| QA-1 Performance, QA-2 Reliability, CON-2 Replaceable AI/NLP Providers                                   | Allows changing AI providers or models with minimal impact; isolates timeouts/errors; keeps Conversation Service technology-agnostic. |
| D2-6  | Use asynchronous queues between Event Intake & Scheduler and Notification Service.                   | UC-2, QA-2 Reliability, QA-1 Performance & Scalability                                                   | Improves robustness against spikes in events and downstream outages; notifications can be retried or delayed without blocking. |

### 7.2 Driver Coverage – Iteration 2

| Driver / Concern                               | Addressed? | How This Iteration Addresses It                                                                                                                |
|-----------------------------------------------|-----------|------------------------------------------------------------------------------------------------------------------------------------------------|
| UC-1: Query Academic Information              | Yes       | Conversation Service + Context & Personalization + AI / NLP Adapter + Integration Adapters define a clear, fast path for academic queries.    |
| UC-2: Receive Notifications and Alerts        | Yes       | Event Intake & Scheduler + Notification Service + NotificationChannel strategies implement an event-driven, multi-channel notification flow. |
| QA-1: Performance & Scalability               | Yes       | Separation of synchronous (Conversation Service) and asynchronous (Notification pipeline) paths; components are stateless and can be scaled.  |
| QA-2: Reliability & Fault Tolerance           | Yes       | Event Intake & Scheduler handles retries/backoff; Notification Service can continue processing queued jobs even if some channels fail.        |
| QA-3: Security & Privacy                      | Yes       | Context & Personalization encapsulates access to user data; subcomponents rely on identifiers and role data from the API Gateway only.        |
| QA-4: Personalization                         | Yes       | Subsystem consistently uses Context & Personalization for both UC-1 answers and UC-2 notifications (preferences, history).                    |
| CON-1: Access via Integration Layer only      | Yes       | All academic and event data flows through LMS/Registration/Calendar/Email Adapters; no direct calls from subcomponents to external systems.   |
| CON-2: Replaceable AI/NLP Providers           | Yes       | AI / NLP Adapter is the single integration point with AI models; configuration and model switching are localized there.                       |
| CON-3: Cloud-Native Deployment Requirement    | Yes       | Subcomponents are designed as independently deployable services (or pods) that fit into the existing AIDAP backend cluster.                   |
