Include in this file the 7 steps for Iteration 3

This iteration focuses on one part of the Core Interaction Subsystem from Iteration 2: the Context & Personalization Service. Here we treat it as its own Context & Personalization Subsystem, and decompose it further to satisfy personalization, security/privacy, and reliability drivers for UC-1 (Query Academic Information) and UC-2 (Receive Notifications & Alerts).

## Step 1 – Use Cases, Quality Attributes, and Constraints

### 1.1 Use Cases (for this element)

#### UC-1C – Provide Context for Academic Query (inside Context & Personalization)

- **Primary actor:** Conversation Service (from Core Interaction Subsystem)
- **Goal:** Obtain the user’s relevant academic and preference context so that the system can answer queries like “When is my next exam?” correctly and in a personalized way.

**Main flow (from this subsystem’s point of view):**

1. Conversation Service calls `getUserContext(userId)` with an authenticated user ID.
2. The Context & Personalization Subsystem loads the **core profile** (enrolments, program, locale) from the User Profile Store.
3. It loads **recent interaction history** (recent questions/answers, last viewed course) from the Conversation History Store.
4. It resolves **preference rules** (e.g., preferred time zone, date format, language, course focus) using stored preferences.
5. It applies **privacy filters**, ensuring only necessary fields are returned for the current request.
6. It returns a **Context DTO** (data transfer object) to the Conversation Service with the minimum information required to answer the query.

**Related requirements (informal mapping):**

- General: R1 (conversational access), R2 (historical interactions), R3 (integrations), R8 (privacy).
- Students: RS1, RS2 (student queries & history), RS5, RS6 (multi-platform & personalization).
- Admins / Maintainers: RA5 (privacy & security policies), RA6 (data retention).
- Quality attributes: QA-4 (Personalization), QA-3 (Security/Privacy).

---

#### UC-2C – Resolve Notification Preferences & Targets

- **Primary actor:** Notification Service (from Core Interaction Subsystem)
- **Goal:** Given an event (e.g., “Assignment due in 24 hours”), determine **which users** should be notified and **how** (channel + timing) in a privacy-respecting way.

**Main flow (from this subsystem’s point of view):**

1. Notification Service calls `getNotificationTargets(event)` with course/event metadata.
2. The Context & Personalization Subsystem queries enrolments and user profiles to find relevant students.
3. It loads each student’s **notification preferences** (channels: chat/push/email; quiet hours; language).
4. It applies **targeting rules** (e.g., only currently enrolled, not withdrawn, opted-in to reminders).
5. It returns a list of **TargetNotificationContext** objects (userId + channel list + locale + any personalization tags).
6. When a notification is sent, Notification Service calls `recordInteraction(userId, notificationMeta)` so history and metrics are updated.

**Related requirements (informal mapping):**

- General: R1, R2, R3.
- Students: RS2, RS3, RS4 (notifications, reminders, schedule changes).
- Admins / Maintainers: RA3, RA4 (analytics and policies), RM4 (log performance metrics).
- Quality attributes: QA-1 (Performance), QA-2 (Reliability), QA-3 (Security & Privacy), QA-4 (Personalization).

---

### 1.2 Quality Attributes (focused on this element)

| ID   | Quality Attribute             | Description (for this subsystem)                                                                                  |
|------|------------------------------|-------------------------------------------------------------------------------------------------------------------|
| QA-1 | Performance & Scalability    | Must return context/preferences for a user in ≤ 100–150 ms under normal load and scale to thousands of users.    |
| QA-2 | Reliability & Fault Tolerance| Must degrade gracefully when downstream data stores (profile/history) are slow or partially unavailable.         |
| QA-3 | Security & Privacy           | Must enforce privacy policies (least-privilege, data minimization, retention limits, role awareness).            |
| QA-4 | Personalization Quality      | Must support fine-grained preferences (channels, timing, locale, course focus) and apply them consistently.      |

---

### 1.3 Constraints (relevant to this element)

| ID    | Constraint Title                        | Description                                                                                       |
|-------|-----------------------------------------|---------------------------------------------------------------------------------------------------|
| CON-4 | Data Access via Approved Stores Only    | May only access user data through **User Profile Store** and **Conversation History Store**.      |
| CON-5 | Policy-Driven Privacy and Retention     | Must respect global **privacy & retention policies** defined by admins (e.g., RA5, RA6, RM2, RM6).|
| CON-6 | Cloud-Native, Stateless Interface Layer | Public API of this subsystem must be stateless and deployable as part of the AIDAP backend.       |
