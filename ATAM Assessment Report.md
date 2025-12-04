# ATAM Evaluation Report  
**AI-Powered Digital Assistant Platform (AIDAP)**

---

# 1. Introduction

For this project, the Architecture Tradeoff Analysis Method (ATAM) was used to evaluate the architectural decisions made thus far for the AI-Powered Digitial Assistant Platform (AIDAP). This report outlines the main quality drivers; the constructed archtecture model and their componenets; QA scenarios; the ATAM utility tree diagram and table; the sensitivity, tradeoff, risk assessment, and non-risk tables; and finally, the summary. The architecture diagram being analyzed comes from the decisions made from iteration 2 in the ADD process.

---

# 2. Business Drivers

## 2.1 Project Goals
The system provides a conversational interface for students, faculty, and administrators to interact with institutional data such as course schedules, deadlines, announcements, and academic analytics. The assistant integrates with external university systems (LMS, registration, calendars, and mail) and uses AI to deliver contextual answers.

## 2.2 Primary Quality Drivers
- **Performance** – response time should be <2 seconds (RS10)  
- **Availability** – 99.5% uptime (RS11), auto-failover (RA6)  
- **Security & Privacy** – SSO authentication, access control, data policies (RS7, RL8, RA5)  
- **Scalability** – up to 5,000 concurrent users (RA7)  
- **Modifiability** – easy integration of new AI models/services (RM5, RM3)  
- **Interoperability** – integrations with LMS, Registration, Calendar, and Email (R3, RD2), and work on mobiles and the web (RS9)

---

# 3. Architectural Approaches

AIDAP uses a client-server architecture.

## 3.1 Architecture Diagram

![Architecture Diagram](images/architecture_diagram.png)


## 3.2 Subcomponents and Responsibilities

| Subcomponent                        | Description / Responsibilities                                                                                                                                         | Layer (logical)          | Related Use Cases |
|------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|-------------------|
| Conversation Service               | Main controller for UC-1. Receives `(userId, message)` from the API Gateway, retrieves user context, calls AI / NLP Adapter for intent and answer, queries Integration Layer adapters, and returns final answer. | Application Services     | UC-1              |
| Notification Service               | Main controller for UC-2. Consumes scheduled notification jobs, determines target users and channels, and dispatches notifications via NotificationChannel strategies.                                        | Application Services     | UC-2              |
| Event Intake & Scheduler           | Listens for events from Integration Layer (LMS/Registration/Calendar). Normalizes events, calculates when notifications should be sent, and queues jobs for Notification Service.                            | Application Services     | UC-2              |
| Context & Personalization Service  | Provides high-level APIs to load user profiles, enrolments, preferences, and history; records interactions/notifications; centralizes personalization rules and privacy enforcement.                         | App Services (with access to Data & Infrastructure) | UC-1, UC-2        |
| AI / NLP Adapter                   | Encapsulates calls to external AI/NLP providers. Offers `interpret(message, context)` and `generateAnswer(data, context)` and handles configuration (model versions, keys), timeouts, and errors.          | Application Services     | UC-1, UC-2 (optional for templated notifications) |
| NotificationChannel strategies     | Interface and concrete channel implementations (ChatChannel, PushChannel, EmailChannel). Used by Notification Service to send messages through different delivery channels.                                | Application Services     | UC-2              |

## 3.3 Subsystem Elements
| Element / Component                | Key Connectors / Interactions                                                                                           |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| Conversation Service              | Receives queries from API Gateway; calls Context & Personalization, AI / NLP Adapter, and LMS/Registration/Calendar Adapters. |
| Notification Service              | Receives jobs from Event Intake & Scheduler; calls Context & Personalization, NotificationChannel strategies, Email Adapter.   |
| Event Intake & Scheduler          | Receives events from LMS/Registration/Calendar Adapters; enqueues notification jobs for Notification Service.           |
| Context & Personalization Service | Communicates with User Profile Store and Conversation History Store; used by Conversation and Notification Services.    |
| AI / NLP Adapter                  | Called by Conversation Service (and optionally Notification Service) to interpret queries and generate text responses.  |
| NotificationChannel strategies    | Implement channel-specific sending (chat, push, email); used by Notification Service.                                   |

## 3.4 Architectural Tactics
- **Performance** – caching, asynchronous calls, load balancing  
- **Availability** – redundancy, health checks, failover  
- **Security** – OAuth SSO, RBAC, encryption  
- **Modifiability** – modular services, model adapter layer  
- **Interoperability** – REST/GraphQL APIs, retry logic  

---

# 4. Quality Attribute Scenarios

### Performance Scenario  
When a student enters a query during normal traffic periods; the system should in <2 seconds.

### Availability Scenario  
If any inference node does fail, the system needs to reroute the traffic to maintain the 99.5 uptime requirement.

### Security Scenario  
Once the user logs in to the application, the system should authenticates using SSO and restricts access to their own info.

### Scalability Scenario  
At 5,000 users, the system should autoscale horizontally without sacraficing performance.

### Modifiability Scenario  
When maintainers deploy a new AI model version, the system should update without causing any downtime.

### Interoperability Scenario  
If the LMS API is unavailable, the system catches errors and tries again.

---

# 5. ATAM Utility Tree

## 5.1 Top-Level Categories
- Performance  
- Availability  
- Security  
- Scalability  
- Modifiability  
- Interoperability  
- Usability  

## 5.2 Utility Tree Diagram

![Utility Tree Diagram](images/utility_tree.png)

## 5.3 Utility Tree Table

| Quality Attribute | Scenario Name | Scenario Description | Stimulus | Response | Priority (Business / Technical) |
|------------------|---------------|----------------------|----------|----------|--------------------------------|
| Performance | P1 | Fast query response | Student sends query | Respond within 2s | H / H |
| Availability | A1 | Failover on component crash | AI node fails | Reroute, maintain 99.5% uptime | H / H |
| Security | S1 | Access control | User logs in | SSO + permissions | H / H |
| Scalability | SC1 | High concurrency | 5000 users active | Auto-scale | H / M |
| Modifiability | M1 | Update AI model | Maintainer deploys update | Zero downtime | M / H |
| Interoperability | I1 | Downstream failure | LMS unavailable | Retry & recover | M / M |
| Usability | U1 | Device adaptability | Student uses mobile | Adaptive UI | M / L |

---

# 6. Scenario Prioritization

### High-Priority Scenarios  
1. Performance (P1)  
2. Availability (A1)  
3. Security (S1)  
4. Scalability (SC1)

### Medium Priority  
- Modifiability (M1)  
- Interoperability (I1)

### Low Priority  
- Usability (U1)

---

# 7. Attribute-Based Analysis  
## 7.1 Sensitivity Points

| Area | Sensitivity Reason |
|------|--------------------|
| AI Model Latency | Complex models/queries increase thinking time |
| API Gateway | Bottleneck affecting availability & scalability |
| Caching Layer | Affects performance |
| SSO Workflow | Impacts security and response speed |
| Autoscaling Rules | Affect scalability and cloud cost |

## 7.2 Tradeoff Points

| Tradeoff | Explanation |
|----------|-------------|
| Performance vs. Accuracy | More accurate models may take a lot more time |
| Caching vs. Freshness | Faster responses may return less accurate data |
| Scalability vs. Cost | Autoscaling increases resource usage and costs |
| Security vs. Usability | Stricter SSO flows reduce convenience |

---

# 8. Risk Assessment Table

| ID | Risk Description | Impact | Probability | Category |
|----|------------------|--------|-------------|----------|
| R1 | Under load, AI models might need >2 seconds to provide an answer | High | Medium | Performance |
| R2 | Failover might not cover all services | High | Low | Availability |
| R3 | External system failures may cascade | Medium | Medium | Interoperability |
| R4 | There may be downtime when models need to be updated | Medium | Medium | Modifiability |
| R5 | Weak authorization may expose data | High | Low | Security |
| R6 | Autoscaling for 5000 users can be costly | Medium | High | Scalability |
| R7 | API gateway might become a single point of failure | High | Medium | Availability |

---

# 9. Non-Risks

| ID | Non-Risk Description |
|----|-----------------------|
| NR1 | Standard APIs ensure interoperability |
| NR2 | SSO + RBAC provide strong security |
| NR3 | Cloud-native deployment supports redundancy |
| NR4 | Logging/monitoring provide operational visibility |

---

# 10. Risk Themes

### 1. AI Inference Bottlenecks  
Latency is highly sensitive and directly impacts performance and scalability.

### 2. External System Reliability  
AIDAP depends heavily on LMS and registration systems.

### 3. Single Points of Failure  
The API gateway is a critical bottleneck and risk.

### 4. Deployment & Modifiability Challenges  
Model updates require reliable CI/CD and versioning.

---

# 11. Summary

The ATAM evaluation shows that AIDAP's architecture can handle the performance, security, and scaling requirements. The utility tree and scenario priorities show the stakeholder needs. The main risks of this architecture are AI latency, failure of external systems due to dependancy, and CI/CD reliability.
