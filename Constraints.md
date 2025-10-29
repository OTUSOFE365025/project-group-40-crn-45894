# System Constraints

> Constraints = limitations/restrictions that narrow design choices

| ID  | Constraint (verbatim/near-verbatim) |
|-----|---|
| R3  | Integrate with existing data sources (registration, LMS, calendars). |
| R4  | Support both text and voice interaction modes. |
| R7  | Deployable as a cloud-native, scalable service. |
| R8  | Protect user data and comply with institutional privacy policies. |
| RS4 | Support multi-language queries and responses. |
| RS7 | Provide secure authentication through the institution’s SSO. |
| RS8 | Ensure student-specific data are visible only to the authenticated user. |
| RS9 | Be accessible on mobile, web, and voice-assistant devices. |
| RS10 | Respond to queries within 2 seconds on average under normal load. |
| RS11 | Remain available 99.5% of the time per month. |
| RS14 | Support offline cache of recent responses for limited connectivity. |
| RL8 | Only authorized lecturers can modify course data. |
| RA5 | Ensure compliance with institutional security and privacy regulations. |
| RA6 | Provide high availability with automatic fail-over and backup recovery. |
| RA7 | Support scalability to handle up to 5,000 concurrent users. |
| RM1 | Deploy updates with zero downtime using continuous deployment pipelines. |
| RM2 | Provide monitoring dashboards (health, latency, errors). |
| RM3 | Support configuration of AI model versions and API keys. |
| RM4 | Log performance metrics for model accuracy and latency. |
| RM5 | Easily extensible to integrate new AI services or external data sources. |
| RM6 | Allow secure backup and restore of user and configuration data. |
| RM7 | Support role-based access for maintenance operations. |
| RD1 | Synchronize data with connected university systems at configurable intervals. |
| RD2 | Use standard APIs (REST or GraphQL) for interoperability. |
| RD3 | Handle failures in data-source availability gracefully (retry and recovery). |
| RD4 | Maintain data integrity and consistency across systems. |
