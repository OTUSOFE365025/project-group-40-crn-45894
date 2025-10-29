# Architectural Concerns

> Concerns = design decisions the team should make whether or not they’re explicitly stated, motivated by the written requirements 

| Concern (what the architecture must address) | Requirement IDs that motivate it | Source(s) |
|---|---|---|
| **External integrations & interoperability** — connectors for LMS/registration/calendars; sync cadence; failure handling; cross-system consistency. | R3, RA1, RD1–RD4 | :contentReference[oaicite:28]{index=28} :contentReference[oaicite:29]{index=29} :contentReference[oaicite:30]{index=30} |
| **Security, privacy & access control** — SSO; per-user visibility; lecturer authorization; institutional compliance. | RS7–RS8, RL8, R8, RA5, RM7 | :contentReference[oaicite:31]{index=31} :contentReference[oaicite:32]{index=32} :contentReference[oaicite:33]{index=33} :contentReference[oaicite:34]{index=34} :contentReference[oaicite:35]{index=35} :contentReference[oaicite:36]{index=36} |
| **Availability & resilience** — uptime target; automatic fail-over; backup recovery; zero-downtime deploys. | RS11, RA6, RM1 | :contentReference[oaicite:37]{index=37} :contentReference[oaicite:38]{index=38} :contentReference[oaicite:39]{index=39} |
| **Performance & scalability** — average latency; concurrency scale; cloud-native deployment model. | RS10, RA7, R7 | :contentReference[oaicite:40]{index=40} :contentReference[oaicite:41]{index=41} :contentReference[oaicite:42]{index=42} |
| **Multi-channel & internationalization** — text+voice; mobile/web/voice-assistant; multi-language; offline cache. | R4, RS4, RS9, RS14 | :contentReference[oaicite:43]{index=43} :contentReference[oaicite:44]{index=44} :contentReference[oaicite:45]{index=45} :contentReference[oaicite:46]{index=46} |
| **Observability & AI operations** — dashboards; model/version & key configuration; accuracy/latency metrics. | RM2–RM4 | :contentReference[oaicite:47]{index=47} |
| **Data protection & recovery** — secure backup/restore for user/configuration data. | RM6 | :contentReference[oaicite:48]{index=48} |

