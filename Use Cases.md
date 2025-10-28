# Use Case Models

## Use Case Overview

The AIDAP system provides a conversational interface for students, faculty, and administrators to interact with
institutional data such as course schedules, deadlines, announcements, and academic analytics.

Each stakeholder performs different functions listed in the next section.

## Primary Actors

| Actor | Description |
| :--- | :--- | :---
| Student (S) | End users who query academic and campus information. |
|Lecturer (L) | Provide course-related content and respond to academic queries |
|Administrator (A) | Maintain institutional data, integrations, and policies. |
|System Maintainer (M) | Responsible for deployment, monitoring, and upgrades. |
| External Data Systems (D) | External systens such as LMS, Registration, Calendar, and Email servers. |

## Main Use Cases

### Use Case 1: Query Academic Information

| Field | Description |
| :--- | :--- |
| Actor | Student |
| Goal | Retrieve academic information such as exam dates, grades, or deadlines. |
| Preconditions | Student should be authenticated through institutional SSO. |
| Main Flow | 