# DAX — Automated Door Access by Reservation

**Vendor:** [Detrios](<!-- TODO: link to Detrios vendor site -->) | **Product:** DAX  
**Maintained by:** Emmanuel (HSD IT/Facilities) | **Last Updated:** <!-- TODO: date -->

---

## Overview

DAX is a licensed product from Detrios that automates physical door access based on facility reservation schedules. Rather than requiring manual intervention to unlock and relock doors for events, DAX bridges the district's facility reservation platform (Facilitron) with the access control system (Lenel OnGuard) — unlocking the right doors at the right time, and relocking them when the reservation ends.

HSD's deployment of DAX is hosted on a dedicated server (`HILLDAX`), managed by the NetOps team. The district's responsibilities are limited to **configuration and integration management** — DAX itself is a a licensed application and its application code is not modified or maintained by district staff.

Alongside DAX, a homegrown **Auto-Notifier** (built and maintained by Emmanuel, running in Retool Workflows) sends renters a daily email listing which doors will be unlocked for their upcoming reservations and for what timeframes.

---

## System Architecture

```
┌─────────────────────┐         ┌──────────────────────────┐
│     Facilitron      │◄───────►│   DAX (on HILLDAX)       │
│  (Reservations)     │  API    │                          │
└─────────────────────┘         │  - Imports reservations  │
                                │  - Applies room→reader   │
┌─────────────────────┐         │    mappings              │
│  Lenel OnGuard      │◄───────►│  - Sends unlock/relock   │
│  (Access Control)   │  OA API │    commands on schedule  │
└─────────────────────┘         └──────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│              Auto-Notifier (Retool Workflow)            │
│                                                         │
│  Runs daily → pulls next-day events from Facilitron →   │
│  applies room→reader mappings → emails renters with     │
│  door access details and timeframes                     │
└─────────────────────────────────────────────────────────┘
```

---

## Components

### Component 1: Detrios DAX

DAX is a licensed access control automation platform. It runs as a service on `HILLDAX` and operates on a polling/schedule basis.

**What it does:**
- Connects to Facilitron via API to import upcoming reservation schedules
- Maps reservations to physical door readers using the Room-to-Reader mapping configuration
- Sends unlock commands to Lenel OnGuard (via the OpenAccess API) at reservation start times
- Sends relock commands at reservation end times

**What HSD manages:**
- DAX configuration (API credentials, room-to-reader mappings, scheduling settings)
- The `HILLDAX` server environment is managed by NetOps

**Relevant links:**
- [DAX Admin Console](http://hilldax/dax)
- [Detrios Support / Vendor Portal](https://detrios.atlassian.net/servicedesk/customer/portal/2)

---

### Component 2: Auto-Notifier (Retool Workflow)

A homegrown daily workflow built and maintained by Emmanuel in Retool. It is independent of DAX but uses the same Facilitron API and room-to-reader mapping logic to generate renter-facing notifications.

**What it does:**
- Runs on a daily schedule (Daily at 10:00am Pacific Time)
- Queries the Facilitron API for all events scheduled for the following day
- Applies the room-to-reader mapping to determine which doors apply to each event
- Sends each renter an email summarizing:
  - Which doors/readers will be unlocked for their reservation
  - The unlock and relock timeframes

**Relevant links:**
- [Retool Workflow — DAX Auto-Notifier](https://hsd1j.retool.com/workflows/a94ae32f-360d-4a1a-81be-2d10820fd0db)

---

## Configuration

### Room-to-Reader Mapping

The core configuration that ties both DAX and the Auto-Notifier together. This mapping defines which Lenel door reader(s) correspond to each Facilitron room/space. Both systems rely on this mapping to determine what physical doors to act on for a given reservation.

- [Room-to-Reader Mapping Document](https://docs.google.com/spreadsheets/d/1DmSXX_WRDscMqNDIQsQpfpLkkl-KvQxGAveR0OU_cZk/edit?usp=sharing)

> **Note:** Any changes to this mapping must be made in all the following places in order to maintain accurate records for both systems to operate correctly.
> - In the internal DAX Room-Reader Mappings, accessed through the DAX admin portal.
> - Retool's internal reader mapping cache (in the Retool Database).
> - On the public spreadsheet linked above. 

---

## Integrations

### Facilitron API
Used by both DAX and the Auto-Notifier to retrieve reservation/event data.

- **API Docs:** [Facilitron Developer Docs](https://facilitron-dev-1c08d20.zuplo.site/api-overview)

### Lenel OnGuard — OpenAccess API
Used exclusively by DAX to send unlock/relock commands to the access control system.

---

## Runbook: Common Tasks

> This section should be expanded as procedures are documented.

| Task | Notes |
|---|---|
| Add a new room-to-reader mapping | Update the mapping doc, then update DAX config and Retool Workflow separately |
| Renter did not receive notification email | Check Retool Workflow run log; verify reservation exists in Facilitron for that date |
| Door did not unlock for a reservation | Check DAX logs on HILLDAX; verify room-to-reader mapping covers that space |
| Door did not relock after reservation | Check DAX logs; may require manual relock via OnGuard |
| Facilitron API errors | Check API status; verify credentials are current |


---

## Ownership & Contacts

| Component | Owner | Notes |
|---|---|---|
| DAX Configuration | Emmanuel Moncada | API config, room-to-reader mappings |
| HILLDAX Server / VM | NetOps | Infrastructure only |
| Auto-Notifier (Retool) | Emmanuel Moncada | Full ownership — code, config, maintenance |
| Facilitron Platform | Denise McMillan / Facilitron | District Facilitron admin contact |
| Lenel OnGuard | Hugo Salmeron / Bosch Building Technologies | District access control admin contact |
