# Quality Attributes

| **ID** | **Name** | **Description** | **Related Requirements** |
|:---|:---|:---|:---|
| QA-1 | Performance | The system delivers conversational responses within ≤2 seconds on average under normal load. Latency is monitored and optimized through scalable AI processing. | RS10, RM2, RM4 |
| QA-2 | Availability | Maintains ≥99.5% uptime monthly through high-availability cloud deployment, automatic failover, and scheduled backups. | RS11, RA6, RM1 |
| QA-3 | Scalability | Supports up to 5,000 concurrent users using elastic cloud infrastructure that auto-scales based on system load. | RA7, R7 |
| QA-4 | Security | Enforces institutional SSO, role-based access, and encryption for sensitive data at rest and in transit. | R8, RS7, RS8, RA5, RM7 |
| QA-5 | Usability | Provides an intuitive conversational interface with multi-language support and cross-platform accessibility (web, mobile, voice). | RS12, RS4, RS9 |
| QA-6 | Maintainability | Enables continuous delivery and rollback of system updates with minimal downtime for maintainers. | RM1, RM2, RM3, RM4 |
| QA-7 | Reliability | Handles data source outages and network errors gracefully using retries, caching, and synchronization mechanisms. | RD3, RD4, RA6 |
| QA-8 | Personalization | Learns from user interactions to improve response accuracy and adapt to individual preferences. | R2, RS5, RS6 |