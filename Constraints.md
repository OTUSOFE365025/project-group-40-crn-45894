# System Constraints

> Constraints = limitations/restrictions that narrow design choices

| FCAPS | ID(s) | Constraint |
|------|-------|------------|
| **F — Fault** | RS11, RA6 | High availability (99.5%/mo), automatic fail-over and backup recovery. |
|  | RM1 | Zero-downtime updates via continuous deployment. |
|  | RM6 | Secure backup and restore of user/configuration data. |
|  | RD3 | Graceful handling of upstream data-source failures (retry/recovery). |
| **C — Configuration** | R3 | Integrate with existing university systems (registration, LMS, calendars). |
|  | RD1 | Synchronize with connected systems at configurable intervals. |
|  | RD2 | Use standard APIs (REST/GraphQL) for interoperability. |
|  | RD4 | Maintain data integrity and consistency across systems. |
|  | R7 | Deployable as cloud-native, scalable service. |
|  | RS4 | Multi-language queries and responses. |
|  | RS9 | Access via mobile, web, and voice-assistant clients. |
|  | RS14 | Offline cache of recent responses for limited connectivity. |
|  | RS7 | Institutional SSO authentication. |
|  | RL8, RS8, RM7 | RBAC: lecturers authorized to modify course data; student data visible only to the authenticated user; maintenance RBAC. |
|  | RM3 | Configure AI model versions and API keys. |
|  | RM5 | Extensible to integrate new AI services or external data sources. |
| **A — Accounting** | RA4 | Monitor system usage and generate analytics reports. |
|  | RM4 | Log evaluation/operational metrics (accuracy, latency). |
|  | RA2 | Define global policies (e.g., retention, response logging). |
| **P — Performance** | RS10 | Average response time ≤ 2 s under normal load. |
|  | RA7 | Scale to ~5,000 concurrent users. |
| **S — Security** | R8, RA5 | Protect user data; comply with institutional privacy/security regulations. |
|  | RS7, RS8, RL8, RM7 | SSO authn; per-user visibility; lecturer authorization; maintenance RBAC. |