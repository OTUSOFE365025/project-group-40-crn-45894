Include in this file the 7 steps for Iteration 3

This iteration focuses on one part of the Core Interaction Subsystem from Iteration 2: the Context & Personalization Service. Here we treat it as its own Context & Personalization Subsystem, and decompose it further to satisfy personalization, security/privacy, and reliability drivers for UC-1 (Query Academic Information) and UC-2 (Receive Notifications & Alerts).

## Step 1 – Use Cases, Quality Attributes, and Constraints

### 1.1 Use Cases (for this element)

#### UC-1C – Provide Context for Academic Query (inside Context & Personalization)

- **Primary actor:** Conversation Service (from Core Interaction Subsystem)
- **Goal:** Obtain the user’s relevant academic and preference context so that the system can answer queries like “When is my next exam?” correctly and in a personalized way.

**Main flow:**

1. Conversation Service calls `getUserContext(userId)` with an authenticated user ID.
2. The Context & Personalization Subsystem loads the **core profile** (enrolments, program, locale) from the User Profile Store.
3. It loads **recent interaction history** (recent questions/answers, last viewed course) from the Conversation History Store.
4. It resolves **preference rules** (e.g., preferred time zone, date format, language, course focus) using stored preferences.
5. It applies **privacy filters**, ensuring only necessary fields are returned for the current request.
6. It returns a **Context DTO** (data transfer object) to the Conversation Service with the minimum information required to answer the query.

**Related requirements :**

- General: R1 (conversational access), R2 (historical interactions), R3 (integrations), R8 (privacy).
- Students: RS1, RS2 (student queries & history), RS5, RS6 (multi-platform & personalization).
- Admins / Maintainers: RA5 (privacy & security policies), RA6 (data retention).
- Quality attributes: QA-4 (Personalization), QA-3 (Security/Privacy).

---

#### UC-2C – Resolve Notification Preferences & Targets

- **Primary actor:** Notification Service (from Core Interaction Subsystem)
- **Goal:** Given an event (e.g., “Assignment due in 24 hours”), determine **which users** should be notified and **how** (channel + timing) in a privacy-respecting way.

**Main flow :**

1. Notification Service calls `getNotificationTargets(event)` with course/event metadata.
2. The Context & Personalization Subsystem queries enrolments and user profiles to find relevant students.
3. It loads each student’s **notification preferences** (channels: chat/push/email; quiet hours; language).
4. It applies **targeting rules** (e.g., only currently enrolled, not withdrawn, opted-in to reminders).
5. It returns a list of **TargetNotificationContext** objects (userId + channel list + locale + any personalization tags).
6. When a notification is sent, Notification Service calls `recordInteraction(userId, notificationMeta)` so history and metrics are updated.

**Related requirements :**

- General: R1, R2, R3.
- Students: RS2, RS3, RS4 (notifications, reminders, schedule changes).
- Admins / Maintainers: RA3, RA4 (analytics and policies), RM4 (log performance metrics).
- Quality attributes: QA-1 (Performance), QA-2 (Reliability), QA-3 (Security & Privacy), QA-4 (Personalization).

---

### 1.2 Quality Attributes 

| ID   | Quality Attribute             | Description                                                                                   |
|------|------------------------------|-------------------------------------------------------------------------------------------------------------------|
| QA-1 | Performance & Scalability    | Must return context/preferences for a user in ≤ 100–150 ms under normal load and scale to thousands of users.    |
| QA-2 | Reliability & Fault Tolerance| Must degrade gracefully when downstream data stores (profile/history) are slow or partially unavailable.         |
| QA-3 | Security & Privacy           | Must enforce privacy policies (least-privilege, data minimization, retention limits, role awareness).            |
| QA-4 | Personalization Quality      | Must support fine-grained preferences (channels, timing, locale, course focus) and apply them consistently.      |

---

### 1.3 Constraints 

| ID    | Constraint Title                        | Description                                                                                       |
|-------|-----------------------------------------|---------------------------------------------------------------------------------------------------|
| CON-4 | Data Access via Approved Stores Only    | May only access user data through **User Profile Store** and **Conversation History Store**.      |
| CON-5 | Policy-Driven Privacy and Retention     | Must respect global **privacy & retention policies** defined by admins (e.g., RA5, RA6, RM2, RM6).|
| CON-6 | Cloud-Native, Stateless Interface Layer | Public API of this subsystem must be stateless and deployable as part of the AIDAP backend.       |

## Step 2 – Element Name and Responsibilities

### 2.1 Selected Element

- **Element name:** Context & Personalization Subsystem  
- **Parent element (from Iteration 2):** Core Interaction Subsystem  
- **External collaborators:**
  - Conversation Service (UC-1: Query Academic Information)
  - Notification Service (UC-2: Receive Notifications & Alerts)
  - User Profile Store
  - Conversation History Store
  - Config & Model Registry
  - Monitoring & Metrics Store

In Iteration 2 we treated this as a single “Context & Personalization Service.”  
In Iteration 3 we zoom in and treat it as its own small subsystem so we can show its internal structure.

---

### 2.2 Responsibilities of the Element

The Context & Personalization Subsystem is responsible for the following:

1. **Provide user context for academic queries**
   - Offer a simple API like `getUserContext(userId)` that the Conversation Service can call.
   - Return the key information about the user (enrolments, program, locale, recent activity) that is needed to answer their academic questions.
   - Supports UC-1C: Provide Context for Academic Query.

2. **Resolve notification targets and preferences**
   - Offer APIs such as `getNotificationTargets(event)` and `getNotificationPreferences(userId)` for the Notification Service.
   - Decide which users should receive a specific notification and which channels to use (chat, push, email), based on enrolments and saved preferences.
   - Supports UC-2C: Resolve Notification Preferences & Targets.

3. **Hide direct access to user profile and history data**
   - Act as the **only** way that the Core Interaction Subsystem reads or writes:
     - Profile, enrolment and preference data (User Profile Store).
     - Interaction history and notification logs (Conversation History Store).
   - Hide database and schema details behind repository-style components so they can be changed later without breaking the rest of the system.

4. **Apply privacy and security rules**
   - Make sure we only return the minimum data needed for the caller (data minimization / least privilege).
   - Respect roles (student, TA, lecturer, admin) when deciding what fields can be exposed.
   - Follow the privacy and data retention policies defined by admins.

5. **Evaluate personalization logic**
   - Combine profile data, preferences, and recent history with configuration rules to:
     - Pick preferred channels, timing and language for notifications.
     - Highlight or prioritize certain courses when answering academic queries (for example, the course the student is currently focused on).
   - Keep all personalization logic in one place so it stays consistent and easier to update.

6. **Support analytics and monitoring**
   - Log anonymized information about context lookups and notification targeting (e.g., success/fail, latency) to the Monitoring & Metrics Store.
   - Record interaction metadata so Analytics & Reporting services can later calculate usage stats and see how well the system is performing.

7. **Expose stateless APIs**
   - Keep the public interface of this subsystem stateless so it can be scaled horizontally in the cloud.
   - Store all long-term data (profiles, history, preferences, configs) in the existing backend stores instead of in memory inside this subsystem.

---

### 2.3 Why This Element Is the Focus of Iteration 3

We chose the Context & Personalization Subsystem for Iteration 3 because:

- It is used by both of the main use cases:
  - UC-1 (Query Academic Information) via UC-1C.
  - UC-2 (Receive Notifications & Alerts) via UC-2C.
- It is the main place where important quality attributes are enforced:
  - **Personalization** (how we tailor answers and notifications per user).
  - **Security and privacy** (who can see what data, and how much).
  - **Performance and reliability** (how quickly and safely we fetch context from the stores).
- It sits directly in front of sensitive user data (profiles, history), so its design strongly impacts compliance with privacy and retention requirements.
- Breaking this part down in more detail makes the overall architecture easier to maintain and extend later (for example, adding new preference types or changing policies without touching the rest of Core Interaction).

## Step 3 – Architectural Patterns and Styles

### 3.1 Overview

In Iteration 3 we are looking inside the **Context & Personalization Subsystem** and choosing the main patterns we will use to structure it.

The idea is to:

- Keep a **simple public interface** for the rest of the system.
- Separate **business rules** (personalization, privacy) from **data access**.
- Make it easier to change storage or policies later without breaking everything.
- Support our key quality attributes: performance, reliability, security/privacy, and personalization.

---

### 3.2 Patterns Used Inside This Subsystem

1. **Facade pattern (external view)**  
   - The subsystem exposes a small set of methods such as:
     - `getUserContext(userId)`
     - `getNotificationTargets(event)`
     - `getNotificationPreferences(userId)`
     - `recordInteraction(userId, metadata)`
   - These are grouped into a **Context Facade API** so the rest of the Core Interaction Subsystem doesn’t need to know about internal components.
   - This keeps the public interface clean and makes it easier to change internals later.

2. **Repository / DAO pattern (data access)**  
   - We use separate repository components:
     - `ProfileRepository` → talks to the **User Profile Store**.
     - `HistoryRepository` → talks to the **Conversation History Store**.
   - These repositories hide the database details (SQL/NoSQL/schema) behind simple methods like `loadProfile(userId)` or `loadRecentInteractions(userId)`.
   - If we ever change how or where the data is stored, we mostly update the repositories, not the rest of the subsystem.

3. **Policy / Rule Engine pattern (personalization + privacy logic)**  
   - A **Preference & Policy Engine** is responsible for:
     - Evaluating user preferences (channels, timing, language, etc.).
     - Applying privacy and retention policies defined by admins.
   - This engine receives raw data from the repositories and configuration from the **Config & Model Registry**, and then outputs a “decision” or “resolved preferences”.
   - This keeps all personalization and policy logic centralized instead of scattering it across many services.

4. **Adapter-style access to external configuration and metrics**  
   - The subsystem talks to:
     - **Config & Model Registry** via a small configuration adapter (e.g., `PolicyConfigAdapter`).
     - **Monitoring & Metrics Store** via an `AuditAndMetricsLogger`.
   - Treating these as adapters makes it easier to change the monitoring or config technology later.

5. **Cross-cutting security/observability components**  
   - **Privacy Guard**:  
     - A small component that all outgoing responses go through.
     - It removes fields that the caller should not see (data minimization, role-based filtering).
   - **Audit & Metrics Logger**:  
     - Logs anonymized access events and latency for operations like `getUserContext` or `getNotificationTargets`.
   - These behave like “sidecar” or cross-cutting concerns: they are reused across multiple flows without putting logging and privacy checks into every method manually.

---

### 3.3 How These Patterns Support Our Drivers

- **Performance & Scalability (QA-1)**
  - Repositories keep data access organized, so we can optimize queries or add caching without changing context-building logic.
  - The Facade API is stateless, so we can run multiple instances of this subsystem in parallel.

- **Reliability & Fault Tolerance (QA-2)**
  - Clear separation between repositories and the rest of the subsystem makes it easier to add retries, timeouts, or fallbacks just around data access.
  - Audit & Metrics Logger helps us detect slow or failing dependencies quickly.

- **Security & Privacy (QA-3)**
  - Privacy Guard ensures that any data leaving the subsystem is filtered according to policy (least privilege and data minimization).
  - Having policies in the **Preference & Policy Engine** keeps privacy rules in one place, which reduces the risk of inconsistent behaviour.

- **Personalization (QA-4)**
  - The Preference & Policy Engine centralizes all personalization rules so answers and notifications are consistent for the same user.
  - Different parts of the system (Conversation Service and Notification Service) can reuse the same engine and get the same personalized behaviour.

Overall, these patterns make the Context & Personalization Subsystem easier to maintain and extend, while still satisfying the main functional and non-functional requirements from Iterations 1 and 2.
