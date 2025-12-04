# ATAM Evaluation Report  
**AI-Powered Digital Assistant Platform (AIDAP)**

---

# 1. Introduction

The Architecture Tradeoff Analysis Method (ATAM) evaluates the architectural decisions made for the AI-Powered Digital Assistant Platform (AIDAP). ATAM identifies risks, non-risks, sensitivity points, and tradeoffs related to quality attributes such as performance, security, availability, scalability, and modifiability. This report applies ATAM to the AIDAP architecture resulting from Iteration 2 (ADD), using stakeholder requirements to generate scenarios, prioritize quality attributes, analyze architectural responses, and document risks.

---

# 2. Business Drivers

## 2.1 Project Goals
AIDAP aims to provide a unified conversational interface for students, lecturers, and administrators to securely access academic data. It must integrate with multiple external university systems, deliver fast and accurate AI-powered responses, and operate as a scalable cloud-native system.

## 2.2 Primary Quality Drivers
- **Performance** – <2 seconds response time (RS10)  
- **Availability** – 99.5% uptime (RS11), auto-failover (RA6)  
- **Security & Privacy** – SSO authentication, access control, data policies (RS7, RL8, RA5)  
- **Scalability** – up to 5,000 concurrent users (RA7)  
- **Modifiability** – easy integration of new AI models/services (RM5, RM3)  
- **Interoperability** – integrations with LMS, Registration, Calendar, and Email (R3, RD2)

---

# 3. Architectural Approaches (From Iteration 2 Architecture)

AIDAP uses a modular cloud-native architecture.

## Architecture Diagram

![Architecture Diagram](images/architecture_diagram.png)


## 3.1 System Decomposition
- **Conversation Interface Layer** – chat UI, voice input, multi-device support  
- **AI Processing Module** – NLU, intent detection, model inference  
- **Integration Layer** – API gateway, connectors to LMS/registration/calendar  
- **Data Management** – user profiles, logs, personalization  
- **Notification Services** – reminders, announcements  
- **Monitoring & Deployment** – CI/CD pipelines, logging, autoscaling  

## 3.2 Architectural Tactics
- **Performance** – caching, asynchronous calls, load balancing  
- **Availability** – redundancy, health checks, failover  
- **Security** – OAuth SSO, RBAC, encryption  
- **Modifiability** – modular services, model adapter layer  
- **Interoperability** – REST/GraphQL APIs, retry logic  

---

# 4. Quality Attribute Scenarios

### Performance Scenario  
A student submits a query during normal load; the system must respond within **2 seconds** on average.

### Availability Scenario  
If an inference node fails, the system automatically reroutes traffic and maintains **99.5% uptime**.

### Security Scenario  
When the user logs in, the system authenticates via SSO and restricts access to their own data.

### Scalability Scenario  
At **5,000 concurrent users**, the system scales horizontally without degraded performance.

### Modifiability Scenario  
When maintainers deploy a new AI model version, the system updates with **zero downtime**.

### Interoperability Scenario  
If the LMS API is unavailable, the system retries gracefully and recovers without user disruption.

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
| AI Model Latency | Complex models increase inference time |
| API Gateway | Bottleneck affecting availability & scalability |
| Caching Layer | Affects performance and data freshness |
| SSO Workflow | Impacts security and response speed |
| Autoscaling Rules | Affect scalability and cloud cost |

## 7.2 Tradeoff Points

| Tradeoff | Explanation |
|----------|-------------|
| Performance vs. Accuracy | More accurate models may exceed response time limits |
| Caching vs. Freshness | Faster responses may return stale data |
| Scalability vs. Cost | Autoscaling increases resource usage |
| Security vs. Usability | Stricter SSO flows reduce convenience |

---

# 8. Risk Assessment Table

| ID | Risk Description | Impact | Probability | Category |
|----|------------------|--------|-------------|----------|
| R1 | AI inference latency may exceed 2s under load | High | Medium | Performance |
| R2 | Failover may not cover all services | High | Low | Availability |
| R3 | External system failures may cascade | Medium | Medium | Interoperability |
| R4 | Model updates may cause downtime | Medium | Medium | Modifiability |
| R5 | Weak authorization may expose data | High | Low | Security |
| R6 | Scaling to 5000 users may exceed budget | Medium | High | Scalability |
| R7 | API gateway may become single point of failure | High | Medium | Availability |

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

The ATAM evaluation shows that AIDAP's architecture supports its performance, security, and scaling requirements. The utility tree and scenario priorities reflect clear stakeholder needs. Main risks involve AI latency, dependency on external systems, and CI/CD reliability. With strong mitigation—redundancy, optimizations, and robust deployment pipelines—AIDAP can meet institutional quality goals.
