# Architectural Concerns

| Concern (what the architecture must address) | Requirement IDs that motivate it |
|---|---|
| External integrations & interoperability — connectors for LMS/registration/calendars; sync cadence; failure handling; cross-system consistency. | R3, RA1, RD1–RD4 |
| Security, privacy & access control — SSO; per-user visibility; lecturer authorization; institutional compliance. | RS7–RS8, RL8, R8, RA5, RM7 |
| Availability & resilience — uptime target; automatic fail-over; backup recovery; zero-downtime deploys. | RS11, RA6, RM1 |
| Performance & scalability — average latency; concurrency scale; cloud-native deployment model. | RS10, RA7, R7 |
| Multi-channel & internationalization — text+voice; mobile/web/voice-assistant; multi-language; offline cache. | R4, RS4, RS9, RS14 |
| Observability & AI operations — dashboards; model/version & key configuration; accuracy/latency metrics. | RM2–RM4 |
| Data protection & recovery — secure backup/restore for user/configuration data. | RM6 |
