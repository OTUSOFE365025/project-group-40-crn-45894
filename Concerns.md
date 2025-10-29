# Architectural Concerns

> Concerns = design decisions the team should make whether or not they’re explicitly stated, motivated by the written requirements

| FCAPS | Concern (what the architecture must address) | Requirement IDs that motivate it |
|------|-----------------------------------------------|----------------------------------|
| **F — Fault** | Resilience & recoverability: meet uptime target; implement fail-over and backup/restore; support zero-downtime deploys. | RS11, RA6, RM1, RM6 |
|  | Fault detection/handling for upstream systems: retries, back-off, health checks. | RD3, RM2 |
| **C — Configuration** | External integrations & interoperability: connectors to LMS/registration/calendars; configurable sync; standard APIs; cross-system consistency. | R3, RD1–RD2, RD4 |
|  | Client/channel capabilities: multi-language; mobile/web/voice-assistant; offline cache. | RS4, RS9, RS14 |
|  | Access & roles: SSO; per-user isolation; lecturer authorization; maintenance RBAC. | RS7, RS8, RL8, RM7 |
|  | Model/service configuration & extensibility: model versions, keys; add new AI services/data sources. | RM3, RM5 |
| **A — Accounting** | Usage/ops analytics and metric collection; retention/logging policy where required. | RA4, RM4, RA2 |
| **P — Performance** | Meet latency and scale objectives; plan capacity for ~5k concurrents; cloud-native scaling strategy. | RS10, RA7, R7 |
| **S — Security** | Institutional privacy/security compliance; authorization boundaries for course and maintenance actions; per-user data protection. | R8, RA5, RS8, RL8, RM7 |