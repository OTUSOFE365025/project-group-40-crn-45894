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