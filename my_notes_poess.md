# Maintenance of Transport Tickets — Team Notes

---

## 1) Simple analysis toolkit — to find real improvements

* **Event list** — write 15–25 key domain events (e.g., *PaymentAuthorized*, *TicketIssued*, *TicketScanned*, *ScanInvalid*, *ServiceDisruption*, *ValidatorHeartbeat*).
* **SIPOC** — Suppliers (banks, DOT/DSB) — Inputs (payment token, QR/NFC) — Process — Outputs (valid ticket, fine/refund) — Customers (riders, back‑office).
* **Pain points** — false “invalid ticket”, slow refunds, device downtime.
* **KPIs** — validation success %, inspection hit rate, device uptime, refund SLA.

---

## 2) The six processes — one‑liners we can model

1. **Ticket Purchase & Issue** — user pays, we issue QR/pass; handle fail/timeout.
2. **Ticket Validation** — scan QR/NFC; accept or deny; log reason.
3. **Ticket Inspection (Control)** — check ticket; if invalid → create case → decide fine.
4. **Refund/Appeal** — open request, auto‑approve simple cases; manual review otherwise.
5. **Validator Device Maintenance** — heartbeat monitoring; create job when device is down.
6. **Disruption Response** — service alert in → notify users and open refund window.

> Keep diagrams one page each — Verb+Noun labels — clear splits/joins — left→right.

---

## 3) Minimal models — just enough to show competence

* **BPMN** — Purchase; Validation; Refund. Use message events + simple service tasks + XOR gateways.
* **DCR** — Inspection case: *CheckTicket* → if invalid then *CreateCase* (response) — *AcceptAppeal* excludes *IssueFine* — *RejectAppeal* includes *IssueFine*.
* **Petri (tiny)** — Validation WF‑net to discuss soundness (optional).

---

## 4) External events & CEP — keep rules tiny

**Event sources (pick two):**

* Transit status — e.g., Realtime trip alerts/updates.
* Weather — rain/snow intensity.

**Domain events (use consistent names):**
`PaymentAuthorized`, `PaymentFailed`, `TicketIssued`, `TicketScanned`, `ScanValid`, `ScanInvalid`, `InspectionStarted`, `FineIssued`, `AppealSubmitted`, `RefundRequested`, `RefundApproved`, `ServiceDisruption`, `ValidatorHeartbeat`.

**Three simple CEP rules:**

1. **Missing Heartbeat** — if no `ValidatorHeartbeat` for 5 min per device → emit `DeviceDown`.
2. **Invalid Spike** — ≥5 `ScanInvalid` at the same stop within 2 min → emit `InspectionNeeded`.
3. **Disruption Trigger** — on transit *Alert(line=X)* → emit `RefundWindowOpen(line=X, t=30m)`.

---

## 5) Integration sketch — how pieces talk

* **CEP → PAIS** — on pattern, send *Correlate Message* to BPMN (key = `ticketId`/`deviceId`).
* **PAIS → CEP** — service tasks emit domain events (*TicketIssued*, *FineIssued*, *RefundApproved*).
* **PAIS → Devices** — mock via MQTT/HTTP — e.g., display *RefundEligible*.

---

## 6) Improvements — small changes that matter

* **Task elimination** — auto‑approve micro‑refunds under a small threshold.
* **Parallelism** — issue QR and send receipt at the same time.
* **Resequence** — validate first, enrich later.
* **Triage** — fast lane for disruption‑tagged cases.

