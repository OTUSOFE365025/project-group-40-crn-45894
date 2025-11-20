Include in this file the 7 steps for Iteration 1

This iteration decomposes the entire AIDAP system into major components/layers to satisfy the drivers from Deliverable 1.

# Step 1:
Element name: AIDAP System (overall platform)

Responsibilities:
AIDAP provides a conversational interface for students, lecturers, administrators, and maintainers to access, manage, and disseminate academic and administrative information. It must:

- Allow students to query academic info (exam dates, grades, deadlines, announcements) via chat/voice.
- Send proactive notifications and alerts (deadlines, schedule changes, announcements).
- Let lecturers publish course materials and announcements, synced with LMS/registration.
- Provide administrators with system analytics (usage, performance, health).
- Support maintainers in deployment, monitoring, uptime, and zero-downtime updates.

# Step 2: Architectural Drivers 

## 2.1: Primary Use Cases
    UC-1: Query Academic Information

        - Actor: Student
        - Goal: Ask questions about grades, deadlines, or exam dates via chat/voice; get accurate contextual answers.
        - Key steps:

            1. Student asks a question in chat/voice.

            2. AI interprets the query.

            3. System retrieves data from LMS/registration/database.

            4. Answer is shown with accurate context.
        - Related requirements: R1, R3, R5, R6, RS1, RS7, RS8, RS10, RS13

    UC-2: Receive Notifications and Alerts

        - Actor: Student
        - Goal: Receive deadline reminders, schedule changes, and announcements.
        - Key steps:

            1. System is triggered by new event/announcement.

            2. Assistant sends a notification (chat/voice).

            3 Student acknowledges/dismisses.
        - Related requirements: R1, R3, RS2, RS6, RS9, RS10

    UC-3: Publish Course Materials and Announcements

        - Actor: Lecturer
        - Goal: Upload/update course content and issue announcements via AIDAP.
        - Key steps:

            1. Lecturer issues a command (e.g., “upload slides”, “post announcement”).

            2. System checks permissions and authorizes.

            3. Data is synchronized with LMS.

            4. Students are notified.
        - Related requirements: R1, R3, RL1, RL2, RL8

    UC-4: Track System Analytics

        Actor: Administrator
        Goal: View usage/performance/health analytics.
        Related requirements: RA4, RA6, RM2, RM4

    UC-5: Do Maintenance and Deployment

        Actor: System Maintainer
        Goal: Push updates and monitor uptime with rollback on failure.
        Related requirements: R7, RA6, RM1, RM2, RM6, RM7

## 2.2: Quality Attributes
    QA-1: Performance

        - Name: Performance
        - Description: The system shall deliver conversational responses within ≤ 2 seconds on average under normal operating load. Performance shall be monitored continuously to ensure compliance with response-time targets.
        - Related requirements: RS10, RM2, RM4 


    QA-2: Availability

        - Name: Availability
        - Description: The system shall maintain ≥ 99.5% monthly uptime through redundant deployment, failover recovery, and scheduled data backups.
        - Related requirements: RS11, RA6, RM1 


    QA-3: Scalability

        - Name: Scalability
        - Description: The system shall support up to 5,000 concurrent users and scale horizontally through elastic cloud infrastructure.
        - Related requirements: RA7, R7


    QA-4: Security

        - Name: Security
        - Description: The system shall enforce institutional SSO, role-based access control, and encryption for all sensitive data in transit and at rest.
        - Related requirements: R8, RS7, RS8, RA5, RM7


    QA-5: Usability

        - Name: Usability
        - Description: The interface shall provide natural conversational interaction with support for multiple languages and cross-platform accessibility (web, mobile, and voice).
        - Related requirements: RS12, RS4, RS9


    QA-6: Maintainability

        - Name: Maintainability
        - Description: The system shall support continuous deployment and rollback of updates with minimal downtime for maintainers.
        - Related requirements: RM1, RM2, RM3, RM4


    QA-7: Reliability

        - Name: Reliability
        - Description: The system shall recover gracefully from data source or network outages using retries, caching, and synchronization.
        - Related requirements: RD3, RD4, RA6


    QA-8: Personalization

        - Name: Personalization
        - Description: The system shall adapt to user behavior by learning from past interactions to improve response accuracy and contextual relevance.
        - Related requirements: R2, RS5, RS6

## 2.3: Constraints (FCAPS)
    F — Fault

        - RS11, RA6: High availability (99.5%/month), automatic fail-over and backup recovery.
        - RM1: Zero-downtime updates via continuous deployment.
        - RM6: Secure backup and restore of user/configuration data.
        - RD3: Graceful handling of upstream data-source failures (retry/recovery).


    C — Configuration

        - R3: Integrate with existing university systems (registration, LMS, calendars).
        - RD1: Synchronize with connected systems at configurable intervals.
        - RD2: Use standard APIs (REST/GraphQL) for interoperability.
        - RD4: Maintain data integrity and consistency across systems.
        - R7: Deployable as cloud-native, scalable service.
        - RS4: Multi-language queries and responses.
        - RS9: Access via mobile, web, and voice-assistant clients.
        - RS14: Offline cache of recent responses for limited connectivity.
        - RS7: Institutional SSO authentication.
        - RL8, RS8, RM7: RBAC (lecturers can modify course data; student data visible only to the authenticated user; maintenance RBAC).
        - RM3: Configure AI model versions and API keys.
        - RM5: Extensible to integrate new AI services or external data sources. 


    A — Accounting

        - RA4: Monitor system usage and generate analytics reports.
        - RM4: Log evaluation/operational metrics (e.g., accuracy, latency).
        - RA2: Define global policies (e.g., retention, response logging).


    P — Performance

        - RS10: Average response time ≤ 2 s under normal load.
        - RA7: Scale to ~5,000 concurrent users.


    S — Security

        - R8, RA5: Protect user data; comply with institutional privacy/security regulations.
        - RS7, RS8, RL8, RM7: SSO authentication; per-user visibility; lecturer authorization;

# Step 3: Reference Architecture

    Architecture style:

        - The AIDAP system uses a cloud-native, layered, service-oriented architecture with:

            - A Presentation / Channel Layer for web, mobile, and voice clients.

            - An API Gateway / Edge Layer as a single secure entry point.

            - An Application Services Layer for conversational logic, notifications, content management, analytics, and ops.

            - An Integration Layer with adapters for LMS, registration, calendar, and email.

            - A Data & Infrastructure Layer for profiles, conversations, configuration, metrics, and backups.

    Rationale (how it supports the drivers):

        - Separating layers improves maintainability and modifiability (QA-6) and isolates concerns.

        - A service-oriented core and adapters support integration constraints and extensibility (R3, RD1–RD4, RM5).

        - Stateless services behind an API gateway enable scalability and performance (QA-1, QA-3, RS10, RA7).

        - API gateway + RBAC + SSO provide a clear security boundary (QA-4, RS7, RS8, RL8, R8, RA5, RM7).

        - Cloud-native deployment with redundancy and backups supports availability and fault tolerance (QA-2, QA-7, RS11, RA6, RM1, RM6).
# Step 4: High-Level Decomposition (Layers & Components)
## 4.1: Presentation / Channel Layer
    Components:

        - Web Chat Client
            - Browser-based chat UI for students, lecturers, and admins.
            - Handles SSO redirect and passes tokens to the gateway.

        - Mobile App Client
            - Native / hybrid mobile app.
            - Supports chat and receives push notifications.

        - Voice Assistant Connector
            - Integrates voice platforms (e.g., smart speaker or voice bot) with AIDAP APIs.

    Responsibilities:

        - Render conversational UI.
        - Collect user input (text/voice) and send it to the backend.
        - Display responses, notifications, and basic status messages.
## 4.2: API Gateway / Edge Layer
    Component:

        - AIDAP API Gateway

    Responsibilities:

        - Single entry point for all clients (web, mobile, voice).
        - Validate SSO tokens and enforce RBAC based on user roles.
        - Route requests to application services.
        - Provide cross-cutting concerns: TLS termination, rate limiting, basic logging, request throttling.
## 4.3: Application Services Layer
    Components:

        - Conversation Orchestrator
            - Coordinates UC-1 (query academic information) and other conversational flows.
            - Calls NLP/AI, retrieves user context, and dispatches to integration services.
            - Applies personalization rules and composes final responses.

        - NLP / AI Adapter
            - Wraps the chosen AI models for NLU and NLG.
            - Supports switching model versions and providers via configuration.

        - Notification Service
            - Implements UC-2 (notifications and alerts).
            - Subscribes to events from LMS/registration/calendar.
            - Schedules and delivers notifications through chat, mobile, or email.

        - Content / Lecturer Service
            - Implements UC-3 (publish/update course materials and announcements).
            - Checks lecturer permissions and calls LMS/registration adapters.
            - Triggers notifications to students after updates.

        - Analytics & Reporting Service
            - Implements UC-4 (system analytics).
            - Aggregates logs and metrics into dashboards for administrators.

        - Maintenance / Ops Console
            - Supports UC-5 (maintenance and deployment).
            - Surfaces health metrics, deployment status, and rollback controls (integrated with CI/CD pipeline).

## 4.4: Integration Layer
    Components:

        - LMS Adapter
            - Connects to LMS via REST/GraphQL.
            - Provides APIs for course content, assignments, and grades.

        - Registration Adapter
            - Connects to the registration system.
            - Provides APIs for enrolments, timetables, and exam schedules.

        - Calendar Adapter
            - Integrates with institutional calendar.
            - Reads and writes events (e.g., deadlines, exam dates) for export (RS13).

        - Email Adapter
            - Integrates with institutional mail system.
            - Used by Notification Service for email-based alerts.

## 4.5: Data & Infrastructure Layer
    Components:

        - User Profile Store
            - Stores user roles, preferences, notification settings, and personalization data.

        - Conversation History Store
            - Logs user queries and system responses for analytics and personalization.

        - Config & Model Registry
            - Stores configuration for AI model versions, API keys, and feature flags.

        - Monitoring & Metrics Store
            - Stores performance metrics (latency, error rates, uptime) and usage statistics.

        - Backup & Recovery
            - Backup jobs, snapshots, and restoration scripts for critical data stores.

# Step 5: Component & Connector View 

## High-level relationships:

    - All clients (web, mobile, voice) call the AIDAP API Gateway.

    - The gateway routes to Conversation Orchestrator, Notification Service, Content / Lecturer Service, Analytics & Reporting Service, and Maintenance / Ops Console.

    - Application services call NLP / AI Adapter, User Profile Store, Conversation History Store, and the Integration Layer adapters (LMS, Registration, Calendar, Email).

    - Analytics and ops components use Monitoring & Metrics Store and Config & Model Registry.

![AIDAP Iteration 1 – High-Level Architecture](<images/Aidap 1 - High level Arch.png>)

# Step 6: Behaviour for Main Scenario (UC-1 Sequence)

## Main scenario: UC-1 – Query Academic Information
    Student asks: “When is my next exam for CPS101?”

![UC-1 – Student Queries Academic Information (Sequence Diagram)](<images/Student queries UC-1 Iter1.png>)

# Step 7: Evaluation & Rationale (How This Satisfies the Drivers)

    - Supports primary use cases (UC-1 to UC-5):

        - Conversation Orchestrator + NLP / AI Adapter + Integration Layer directly implement UC-1.

        - Notification Service + Calendar/Email Adapters implement UC-2.

        - Content / Lecturer Service + LMS/Registration Adapters implement UC-3.

        - Analytics & Reporting Service + Monitoring & Metrics Store implement UC-4.

        - Maintenance / Ops Console + CI/CD integration support UC-5.

    - Meets quality attributes:

        - Performance & Scalability (QA-1, QA-3): stateless services, API gateway, and cloud-native scaling.

        - Availability & Reliability (QA-2, QA-7): redundancy, backups, and graceful handling of upstream failures.

        - Security (QA-4): SSO, RBAC, and encryption enforced at the gateway and within services.

        - Maintainability (QA-6): modular services and clear layering support continuous deployment and localized changes.

        - Personalization (QA-8): User Profile Store and Conversation History Store enable adaptive responses and targeted notifications.

    - Respects constraints (FCAPS):

        - Integrates with LMS, registration, calendar, and email via adapters.

        - Uses standard APIs, cloud-native deployment, and centralized monitoring.

        - Implements accounting, performance, and security constraints explicitly in the architecture.