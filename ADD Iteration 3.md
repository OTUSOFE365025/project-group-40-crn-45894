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


## Step 4 – Decomposition of the Context & Personalization Subsystem

In this step we break the **Context & Personalization Subsystem** into smaller internal components and explain what each one does and how they relate to Iterations 1 and 2.

---

### 4.1 Subcomponents and Responsibilities

Below are the main subcomponents we introduce inside this subsystem.

| Subcomponent                 | Main Responsibilities                                                                                               |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------|
| Context Facade API          | Public entry point for other services (Conversation, Notification). Exposes methods like `getUserContext` and `getNotificationTargets`. |
| ProfileRepository           | Handles all reads of user profiles, enrolments, and static preferences from the **User Profile Store**.             |
| HistoryRepository           | Handles reads and writes of interaction history and notification logs in the **Conversation History Store**.       |
| Preference & Policy Engine  | Applies personalization rules (channels, timing, locale) and privacy/retention policies based on configs + user data. |
| Context Assembler           | Orchestrates calls to repositories and the engine to build the final **Context DTO** returned by `getUserContext`. |
| Notification Target Resolver| Orchestrates the logic for `getNotificationTargets(event)` and builds the list of users + channels for notifications. |
| Privacy Guard               | Applies data minimization and role-based filtering before anything leaves the subsystem.                            |
| Audit & Metrics Logger      | Sends anonymized logs and metrics (latency, success/failure) to the **Monitoring & Metrics Store**.                |

A quick summary of each:

1. **Context Facade API**
   - This is the “front door” of the subsystem.
   - Called by:
     - Conversation Service (for user context).
     - Notification Service (for targets and preferences).
   - It doesn’t do heavy work itself; it just validates inputs and delegates to the right internal component (Context Assembler or Notification Target Resolver).

2. **ProfileRepository**
   - Talks to the **User Profile Store**.
   - Typical responsibilities:
     - `loadProfile(userId)`
     - `loadEnrolledUsers(courseId)`
     - `loadStaticPreferences(userId)`
   - It hides all database details (schemas, queries) from the rest of the subsystem.

3. **HistoryRepository**
   - Talks to the **Conversation History Store**.
   - Typical responsibilities:
     - `loadRecentInteractions(userId, limit)`
     - `recordInteraction(userId, metadata)`
     - `recordNotification(userId, notificationMeta)`
   - This is where we centralize access to historical data to keep it consistent.

4. **Preference & Policy Engine**
   - The “brain” for personalization and policies.
   - Inputs:
     - Raw profile data (from `ProfileRepository`).
     - Interaction history (from `HistoryRepository`).
     - Policy and preference configs (from the Config & Model Registry).
   - Outputs:
     - Resolved preferences (channels, timing, locale).
     - Flags about what data can or cannot be exposed (for Privacy Guard).
   - This is where we check admin policies (e.g., retention limits, opt-in/out rules).

5. **Context Assembler**
   - Used mainly for **UC-1C (Provide Context for Academic Query)**.
   - Steps (simplified):
     1. Calls `ProfileRepository` to get basic user info and enrolments.
     2. Calls `HistoryRepository` to get recent interactions.
     3. Passes everything to the `Preference & Policy Engine` to resolve personalization details.
     4. Sends the result through `Privacy Guard`.
     5. Logs the operation via `Audit & Metrics Logger`.
   - Returns a final `ContextDTO` to the **Context Facade API**.

6. **Notification Target Resolver**
   - Used mainly for **UC-2C (Resolve Notification Preferences & Targets)**.
   - Steps (simplified):
     1. Uses `ProfileRepository` to find all students enrolled in the relevant course(s).
     2. For each user, asks the `Preference & Policy Engine` to resolve notification preferences.
     3. Sends target data through the `Privacy Guard`.
     4. Logs the operation via `Audit & Metrics Logger`.
   - Returns a list of **TargetNotificationContext** objects to the **Context Facade API**.

7. **Privacy Guard**
   - Sits on the boundary of the subsystem.
   - Before any context or target list is returned:
     - Removes fields the caller shouldn’t see.
     - Checks role and purpose (e.g., student vs lecturer vs admin).
   - Helps us enforce “least privilege” and data minimization.

8. **Audit & Metrics Logger**
   - Used by both Context Assembler and Notification Target Resolver (and potentially other flows later).
   - Logs:
     - Operation type (e.g., `getUserContext`, `getNotificationTargets`).
     - High-level status (success/failure).
     - Latency/response times.
   - Sends this information to the **Monitoring & Metrics Store** so that Admins/Maintainers can track health and performance.

---

### 4.2 Mapping Back to Iterations 1 and 2

To show continuity with earlier iterations:

- In **Iteration 1** we only had:
  - **User Profile Store**, **Conversation History Store**, **Config & Model Registry**, and **Monitoring & Metrics** as big blocks in the Data & Infrastructure layer.
- In **Iteration 2** we introduced:
  - A single **Context & Personalization Service** inside the **Core Interaction Subsystem**.

Now, in **Iteration 3**:

- That single “Context & Personalization Service” from Iteration 2 is realized by the set of subcomponents above:
  - `Context Facade API`
  - `Context Assembler`
  - `Notification Target Resolver`
  - `ProfileRepository`
  - `HistoryRepository`
  - `Preference & Policy Engine`
  - `Privacy Guard`
  - `Audit & Metrics Logger`

So, when we draw the Component & Connector (C&C) diagram for this iteration, we will show:

- **From outside:**
  - Conversation Service → Context Facade API
  - Notification Service → Context Facade API

- **Inside this subsystem:**
  - Context Facade API → Context Assembler (for UC-1C)
  - Context Facade API → Notification Target Resolver (for UC-2C)
  - Orchestrators → Repositories → Data Stores
  - Orchestrators → Preference & Policy Engine → Config & Model Registry
  - Orchestrators → Privacy Guard → back to Facade
  - Orchestrators → Audit & Metrics Logger → Monitoring & Metrics Store

This decomposition makes it clear how the Context & Personalization Subsystem actually works internally while still matching the higher-level design from Iterations 1 and 2.

## Step 5 – Component & Connector View (Context & Personalization Subsystem)

Figure 5.1 shows the Component & Connector (C&C) view for the **Context & Personalization Subsystem**.  
The diagram file is stored as: `images/Iteration 3 C&C.png`.

![Context & Personalization Subsystem – Component & Connector View](<images/Iteration 3 C&C.png>)

---

### 5.1 Scope and Context

At this level we focus only on the **Context & Personalization Subsystem** and how it connects to:

- The rest of the **Core Interaction Subsystem** (Conversation Service and Notification Service).
- The main data/infra components from Iteration 1:
  - User Profile Store  
  - Conversation History Store  
  - Config & Model Registry  
  - Monitoring & Metrics Store  

In the diagram, the Core Interaction Subsystem is shown at the top, the Context & Personalization Subsystem is in the middle box, and the data stores are at the bottom.

---

### 5.2 External Connectors

**Incoming connections (from Core Interaction):**

- **Conversation Service → Context Facade API**
  - Calls:
    - `getUserContext(userId)`
    - `recordInteraction(userId, metadata)`

- **Notification Service → Context Facade API**
  - Calls:
    - `getNotificationTargets(event)`
    - `getNotificationPreferences(userId)`
    - `recordInteraction(userId, notificationMeta)`

**Outgoing connections (to infrastructure):**

- **ProfileRepository → User Profile Store**
  - Reads:
    - User profile data
    - Enrolments
    - Static preferences

- **HistoryRepository → Conversation History Store**
  - Reads:
    - Recent interactions
  - Writes:
    - Interaction history
    - Notification logs

- **Preference & Policy Engine → Config & Model Registry**
  - Reads:
    - Privacy and retention policies
    - Personalization configuration (default channels, feature flags, etc.)

- **Audit & Metrics Logger → Monitoring & Metrics Store**
  - Writes:
    - Metrics for operations like `getUserContext` and `getNotificationTargets`
    - High-level/anonymized access logs

---

### 5.3 Internal Components and Connectors

Inside the subsystem (inside the big box in Figure 5.1), the main components and their connectors are:

| From                          | To                             | Purpose                                                                                       |
|-------------------------------|--------------------------------|-----------------------------------------------------------------------------------------------|
| Context Facade API           | Context Assembler              | Handle `getUserContext(userId)` (UC-1C).                                                      |
| Context Facade API           | Notification Target Resolver   | Handle `getNotificationTargets(event)` (UC-2C).                                               |
| Context Assembler            | ProfileRepository              | Load user profile, enrolments and static preferences.                                         |
| Context Assembler            | HistoryRepository              | Load recent interaction history for the user.                                                |
| Context Assembler            | Preference & Policy Engine     | Ask for resolved context and personalization decisions.                                      |
| Context Assembler            | Privacy Guard                  | Filter the final Context DTO before returning it.                                            |
| Context Assembler            | Audit & Metrics Logger         | Log `getUserContext` latency and success/failure.                                            |
| Notification Target Resolver | ProfileRepository              | Load all enrolled users for a course or event.                                               |
| Notification Target Resolver | Preference & Policy Engine     | Resolve notification preferences (channels, timing, opt-in/out) for each user.               |
| Notification Target Resolver | Privacy Guard                  | Filter the final list of targets before returning it.                                        |
| Notification Target Resolver | Audit & Metrics Logger         | Log `getNotificationTargets` latency and success/failure.                                    |
| HistoryRepository            | Conversation History Store     | Read/write interaction and notification history.                                             |
| ProfileRepository            | User Profile Store             | Read user profile and enrolment data.                                                        |
| Preference & Policy Engine   | Config & Model Registry        | Load the current set of policies and personalization rules.                                  |
| Audit & Metrics Logger       | Monitoring & Metrics Store     | Send anonymized logs and metrics for monitoring and analytics.                               |

---

### 5.4 Text Description of the Diagram

The typical flow in Figure 5.1 looks like this:

1. A caller (Conversation Service or Notification Service) sends a request to the **Context Facade API**.
2. The Facade delegates the request to either:
   - **Context Assembler** (for `getUserContext`), or  
   - **Notification Target Resolver** (for `getNotificationTargets`).
3. The orchestrator component calls the **repositories** (`ProfileRepository`, `HistoryRepository`) to get the required data.
4. It then calls the **Preference & Policy Engine** to apply personalization and policy rules.
5. Before anything leaves the subsystem, the result passes through the **Privacy Guard** to enforce data minimization and role-based filtering.
6. Throughout the process, the **Audit & Metrics Logger** records metrics and status and sends them to the Monitoring & Metrics Store.
7. Finally, the **Context Facade API** returns the filtered result back to the caller.

This C&C view shows how all the internal parts of the Context & Personalization Subsystem work together and how they connect back to the components introduced in Iterations 1 and 2.

## Step 6 – Behaviour for Main Scenarios

In this step we explain how the **Context & Personalization Subsystem** behaves in its two main internal use cases:

- UC-1C – Provide Context for Academic Query  
- UC-2C – Resolve Notification Preferences & Targets  

We also include two sequence diagrams from the `images` folder:

- `images/Iteration 3 UC-1C Sequence.png`
- `images/Iteration 3 UC-2C Sequence.png`

---

### 6.1 UC-1C – Provide Context for Academic Query

**Scenario summary:**  
A student types a question like *“When is my next exam for CPS101?”* into the chat. The Conversation Service needs user-specific context (enrolments, preferences, recent activity), so it calls the Context & Personalization Subsystem.

![UC-1C – Provide Context for Academic Query](images/Iteration%203%20UC-1C%20Sequence.png)

**Step-by-step behaviour (from this subsystem’s point of view):**

1. **Conversation Service → Context Facade API**  
   - The Conversation Service calls `getUserContext(userId)` with the authenticated `userId`.

2. **Context Facade API → Context Assembler**  
   - The Facade does basic validation and forwards the request to `Context Assembler` using something like `buildUserContext(userId)`.

3. **Context Assembler → ProfileRepository**  
   - `Context Assembler` calls `loadProfile(userId)`.  
   - `ProfileRepository` reads from the **User Profile Store** and returns:
     - Basic profile data (program, year, etc.).
     - Current course enrolments (e.g., CPS101, MATH2050).
     - Static preferences (time zone, date format, language).

4. **Context Assembler → HistoryRepository**  
   - The assembler calls `loadRecentInteractions(userId, limit)` on `HistoryRepository`.  
   - `HistoryRepository` reads recent interaction history from the **Conversation History Store** and returns, for example, the last few questions the student asked.

5. **Context Assembler → Preference & Policy Engine**  
   - The assembler now calls `resolveContext(profile, interactions, configRef)` on the **Preference & Policy Engine**.  
   - The engine:
     - Pulls policies and personalization rules from the **Config & Model Registry**.
     - Infers a “focus course” (e.g., CPS101) based on recent activity.
     - Applies user preferences (language, date format, etc.).
     - Applies any high-level rules (for example, hide withdrawn courses).
   - It returns a `resolvedContextModel` back to the assembler.

6. **Context Assembler → Privacy Guard**  
   - The assembler passes the `resolvedContextModel` to the **Privacy Guard** with the caller role (`"ConversationService"`).  
   - The Privacy Guard removes any fields the caller does not need (internal IDs, extra flags) and returns a filtered `ContextDTO`.

7. **Context Assembler → Audit & Metrics Logger**  
   - The assembler logs the operation using `logContextLookup(userId*, latency, status)` on the **Audit & Metrics Logger**.  
   - Only an anonymized identifier is used here (no raw personal data).

8. **Context Assembler → Context Facade API → Conversation Service**  
   - The `ContextDTO` is passed back to the **Context Facade API**, which then returns it to the **Conversation Service**.  
   - The Conversation Service uses this context, plus data from the LMS/Registration and the AI/NLP Adapter (from Iteration 2), to build the final answer for the student.

This flow shows how UC-1C supports the larger UC-1 (Query Academic Information) while still enforcing personalization and privacy rules.

---

### 6.2 UC-2C – Resolve Notification Preferences & Targets

**Scenario summary:**  
The LMS generates an event like *“CPS101 Assignment 2 is due in 24 hours.”* The Notification Service needs to know which students to notify and which channels to use, while respecting their notification preferences and quiet hours.

![UC-2C – Resolve Notification Preferences & Targets](images/Iteration%203%20UC-2C%20Sequence.png)

**Step-by-step behaviour :**

1. **Notification Service → Context Facade API**  
   - The Notification Service calls `getNotificationTargets(event)` on the **Context Facade API**, passing information such as:
     - `courseId = CPS101`
     - `dueDateTime`
     - `notificationType = AssignmentReminder`.

2. **Context Facade API → Notification Target Resolver**  
   - The Facade forwards the event to the **Notification Target Resolver** using `resolveTargets(event)`.

3. **Notification Target Resolver → ProfileRepository**  
   - The resolver calls `loadEnrolledUsers(courseId)` on `ProfileRepository`.  
   - `ProfileRepository` queries the **User Profile Store** and returns a list of `userId`s for all students currently enrolled in CPS101.

4. **Notification Target Resolver → Preference & Policy Engine (loop per user)**  
   - For each enrolled user, the resolver calls  
     `resolveNotificationPrefs(userId, eventType, dueDateTime)` on the **Preference & Policy Engine**.
   - The engine:
     - Reads notification and quiet-hour policies from the **Config & Model Registry**.
     - Looks up the user’s notification preferences (channels, opt-in/out flags, language).
     - Uses the user’s time zone to decide if sending now is allowed (quiet hours).
     - Returns a “decision” object that says:
       - Which channels are allowed (chat/email/push).
       - Whether sending now is allowed.
       - Whether the user has opted in for this event type.

5. **Notification Target Resolver – filter targets**  
   - The resolver filters the list of users:
     - Removes users who have opted out of this notification type.
     - Removes users who cannot be contacted now because of quiet-hour rules.
   - For the remaining users, it builds `TargetNotificationContext` entries with:
     - `userId`
     - allowed channels
     - locale/language and any simple personalization flags.

6. **Notification Target Resolver → Privacy Guard**  
   - The resolver calls `filterTargets(targetList, callerRole="NotificationService")` on the **Privacy Guard**.  
   - The Privacy Guard strips out any unnecessary or sensitive details so only the minimum data required to send the notifications is returned.

7. **Notification Target Resolver → Audit & Metrics Logger**  
   - The resolver logs the operation using `logTargetResolution(targetCount, latency, status)` on the **Audit & Metrics Logger**.

8. **Notification Target Resolver → Context Facade API → Notification Service**  
   - The final `filteredTargetList` is passed back to the **Context Facade API**, and then returned to the **Notification Service**.  
   - The Notification Service then uses its notification channel strategies (from Iteration 2) to actually send the reminders via chat, email, and/or push.

This flow shows how UC-2C supports UC-2 (Receive Notifications & Alerts) by figuring out **who** should be notified and **how**, while still respecting personalization and privacy requirements.

## Step 7 – Design Decisions and Driver Coverage

In this step we list the main design decisions we made for the **Context & Personalization Subsystem** in Iteration 3, and explain how they relate back to the use cases and quality attribute drivers from Iterations 1 and 2.

---

### 7.1 Key Design Decisions

| ID   | Design Decision | Short Explanation |
|------|-----------------|------------------|
| D3-1 | **Use a single Context Facade API as the only entry point.** | Conversation Service and Notification Service never talk directly to repositories or engines. They always go through `Context Facade API` using methods like `getUserContext` and `getNotificationTargets`. |
| D3-2 | **Introduce ProfileRepository and HistoryRepository as separate data access layers.** | All reads/writes to the User Profile Store and Conversation History Store are done through repository components instead of scattered queries. |
| D3-3 | **Create a dedicated Preference & Policy Engine.** | Personalization rules (channels, timing, locale) and privacy/retention policies are evaluated in one place instead of being hard-coded across services. |
| D3-4 | **Add a Privacy Guard on the subsystem boundary.** | Before any context or target list leaves the subsystem, it passes through Privacy Guard to enforce data minimization and role-based filtering. |
| D3-5 | **Split orchestration into Context Assembler and Notification Target Resolver.** | UC-1C and UC-2C each get their own orchestrator component instead of one huge “god service”, which keeps logic focused and easier to reason about. |
| D3-6 | **Log operations via an Audit & Metrics Logger.** | Every major operation (`getUserContext`, `getNotificationTargets`) records latency and status to Monitoring & Metrics so admins can track health and performance. |

---

#### D3-1 – Context Facade API as the only entry point

- **Why we did it:**  
  We wanted other services to see this subsystem as a simple “black box”: they just call a small set of methods and don’t care how the internals work.
- **What it helps with:**  
  - Makes the subsystem easier to change later without breaking Conversation or Notification Services.  
  - Helps security: all requests come through one place, so validation and access checks are simpler.

#### D3-2 – Separate repositories for profile and history data

- **Why we did it:**  
  Profile/enrolment data and interaction history have different access patterns and could end up in different storage technologies. Keeping them in two repositories matches that.
- **What it helps with:**  
  - Easier to tune performance separately (e.g., caching profile lookups without touching history).  
  - If the university changes how profiles are stored, we mostly update `ProfileRepository` instead of touching the whole subsystem.

#### D3-3 – Preference & Policy Engine

- **Why we did it:**  
  Personalization rules and privacy/retention policies can get complicated and will probably change over time. We didn’t want those scattered across multiple services.
- **What it helps with:**  
  - Keeps personalization behaviour consistent between UC-1C and UC-2C.  
  - Makes it easier to update policies (for example, new opt-in rules) by changing one component.

#### D3-4 – Privacy Guard on the boundary

- **Why we did it:**  
  We want to enforce “least privilege” and “data minimization” by default. It’s easy to accidentally leak extra fields if we don’t have a dedicated filter step.
- **What it helps with:**  
  - Stronger alignment with privacy requirements (R8, RA5, RA6).  
  - Makes audits easier: we can point to one place that controls what leaves the subsystem.

#### D3-5 – Separate orchestrators for UC-1C and UC-2C

- **Why we did it:**  
  The flow for building query context is not the same as the flow for resolving notification targets. Putting both into one big class/service would be messy.
- **What it helps with:**  
  - Clearer behaviour: Context Assembler is only about `getUserContext`, Notification Target Resolver is only about `getNotificationTargets`.  
  - Better maintainability: changes for notifications don’t risk breaking query context, and vice versa.

#### D3-6 – Audit & Metrics Logger

- **Why we did it:**  
  The project requirements talk about monitoring, analytics, and maintainability. To support that, we need structured metrics about how this subsystem behaves.
- **What it helps with:**  
  - Detecting performance problems (e.g., AI or database slowdowns).  
  - Supporting admin dashboards and future ATAM-style evaluations.

---

### 7.2 Driver Coverage

The table below shows how these decisions relate back to the main drivers (use cases, quality attributes, and constraints).

| Driver / Concern | How Iteration 3 addresses it |
|------------------|------------------------------|
| **UC-1 – Query Academic Information (via UC-1C)** | D3-1 and D3-5 make `getUserContext` a clean operation through the Context Facade API and Context Assembler. D3-2 and D3-3 ensure profile/history data and personalization rules are handled properly so the Conversation Service always gets a useful ContextDTO. |
| **UC-2 – Receive Notifications & Alerts (via UC-2C)** | D3-1 and D3-5 support `getNotificationTargets` through the Facade and Notification Target Resolver. D3-2 and D3-3 provide accurate, per-user channel and timing preferences. D3-4 ensures that only the minimum data needed to send notifications is returned. |
| **QA-1 – Performance & Scalability** | D3-2 separates data access into repositories so we can optimize queries and add caching where needed. D3-1 and the stateless Facade design mean we can scale this subsystem horizontally. D3-6 provides metrics to spot performance bottlenecks. |
| **QA-2 – Reliability & Fault Tolerance** | D3-2 makes it easier to add retries/timeouts around the repositories only. D3-6 gives visibility into failures and slow operations, which helps maintainers react quickly. Using a single Facade (D3-1) also simplifies error handling from the callers’ point of view. |
| **QA-3 – Security & Privacy** | D3-4 (Privacy Guard) and D3-3 (policy engine) are the main decisions for this. They enforce data minimization, role-based filtering, and retention/privacy rules before data leaves the subsystem. D3-1 helps by centralizing access. |
| **QA-4 – Personalization** | D3-3 centralizes all preference and policy logic, so both queries and notifications use the same personalization rules. D3-5 ensures each use case can apply personalization in the way that makes the most sense for that flow. |
| **Constraints (CON-4, CON-5, CON-6)** | D3-1 and D3-2 guarantee that all access to user/profile data goes through approved stores and a stateless API. D3-3 and D3-4 ensure policy-driven privacy and retention are actually enforced inside the subsystem. |

---

### 7.3 Short Summary

Overall, Iteration 3 takes the “Context & Personalization Service” from Iteration 2 and turns it into a small, well-structured subsystem.  

The main ideas are:

- One Facade in front,  
- Repositories for data access,  
- A central engine for personalization/policies,  
- A Privacy Guard at the boundary,  
- Two focused orchestrators for the main flows, and  
- Logging for monitoring.

These choices keep the subsystem aligned with the earlier architecture while making personalization and privacy much easier to manage and evolve in the future.
