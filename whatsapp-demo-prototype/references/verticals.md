# Vertical Step Sequence Templates

Pre-built step sequences for the verticals 8x8 sells into most often in APAC. Pick the closest match and adapt — don't write from scratch unless the use case is genuinely novel.

Each template lists the 6–10 step sequence with title, type, category, and trigger. Copy the structure into `DEMO_DATA[market].steps` and fill in `preview`, `params`, `buttons`, and brand-specific content.

---

## Banking — Credit Card / Loan Acquisition

The default banking acquisition pattern. Use for any "acquire a card / loan / account" journey.

| # | Title | Type | Category | Trigger |
|---|---|---|---|---|
| 1 | Pre-Approval Offer | template | MARKETING | CRM pre-approval engine flags eligible customer |
| 2 | Customer Replies "Apply Now" | customer_reply | REPLY | Customer taps quick-reply button → triggers Flow |
| 3 | Application Received | template | UTILITY | Application submitted via WhatsApp Flow |
| 4 | Identity Verification | template | UTILITY | KYC requires identity check (Singpass / SMS OTP / KTP scan) |
| 5 | Verification Complete | system_event | SYSTEM | Identity provider callback confirms |
| 6 | Application Approved | template | UTILITY | Credit decisioning engine approves |
| 7 | Card Activation | template | UTILITY | Customer activates card |
| 8 | Payment Reminder | template | UTILITY | Billing system detects upcoming due date |
| 9 | Service Hub | template | UTILITY | Customer opens service menu (Speak to Agent → Moobidesk) |

**Variants**: For loans, swap step 1 to "Loan Offer", step 6 to "Loan Approved with rate", step 8 to "Repayment Reminder". For deposit accounts, drop credit decisioning and add an "Initial Deposit Confirmation" step.

---

## Banking — Customer Servicing (post-acquisition)

For demos focused on existing customers (account inquiries, transaction alerts, dispute handling).

| # | Title | Type | Category | Trigger |
|---|---|---|---|---|
| 1 | Transaction Alert | template | UTILITY | Card swipe exceeds notification threshold |
| 2 | Customer Confirms / Disputes | customer_reply | REPLY | Customer taps "Yes, that's me" or "I don't recognize this" |
| 3 | Dispute Initiated | template | UTILITY | Fraud workflow opens case |
| 4 | Card Block Confirmation | template | UTILITY | Card status updated to BLOCKED |
| 5 | Replacement Card Dispatched | template | UTILITY | Replacement card workflow triggers |
| 6 | Speak to Agent → Moobidesk | template | SERVICE | Customer requests human assistance |

---

## Banking — CRM Bulk Reminders (ops-facing, broadcast + inbox)

For demos where the audience is the collections / ops team, not the customer. Typically pairs broadcast mode with an embedded Moobidesk stage.

| # | Stage | Mode | Purpose |
|---|---|---|---|
| 1 | Today's Batch | workflow (left: ops console, right: WhatsApp preview) | Review segmented recipient list + template params |
| 2 | Campaign Live | broadcast (recipient table with per-row status animation) | Fire the bulk send; watch delivery/read/reply states |
| 3 | Reply Inbox | embedded Moobidesk | Triage inbound replies with auto-categorisation + smart-reply chips |

See `references/moobidesk-inbox.md` for the stage 3 pattern and `references/broadcast.md` for stage 2.

---

## Insurance — Policy Renewal + Claims

| # | Title | Type | Category | Trigger |
|---|---|---|---|---|
| 1 | Renewal Reminder (60 days) | template | UTILITY | Policy expiry approaching |
| 2 | Customer Taps "Renew Now" | customer_reply | REPLY | → triggers Flow with policy options |
| 3 | Renewal Quote | template | UTILITY | Pricing engine returns updated premium |
| 4 | Payment Confirmation | template | UTILITY | Payment gateway returns success |
| 5 | Updated Policy Document | template | UTILITY | Document service generates PDF (link out) |
| 6 | Submit Claim | template | UTILITY | Customer initiates claims journey |
| 7 | Photo Upload via WhatsApp | customer_reply | REPLY | Customer sends image (rendered as image bubble) |
| 8 | Claim Status Update | template | UTILITY | Claims engine updates status |
| 9 | Claim Approved + Payout | template | UTILITY | Final disposition |

**Note**: Photo upload simulation requires an extra step type (`media_message`) — use a static placeholder image and render as an outgoing bubble with `<img>` content.

---

## Retail / E-commerce — Order Lifecycle + Loyalty

| # | Title | Type | Category | Trigger |
|---|---|---|---|---|
| 1 | Cart Abandonment Nudge | template | MARKETING | Customer left items in cart >2 hours |
| 2 | Customer Taps "Complete Order" | customer_reply | REPLY | → opens checkout link |
| 3 | Order Confirmation | template | UTILITY | Order placed in OMS |
| 4 | Shipping Notification | template | UTILITY | Carrier API returns tracking number |
| 5 | Out for Delivery | template | UTILITY | Last-mile carrier scan |
| 6 | Delivered + Rate Experience | template | UTILITY | Delivery confirmation triggers post-purchase survey |
| 7 | Loyalty Points Earned | template | UTILITY | Loyalty engine credits points |
| 8 | Personalized Re-engagement | template | MARKETING | Recommendation engine flags lookalike products |

---

## Telco — Onboarding + Plan Management

| # | Title | Type | Category | Trigger |
|---|---|---|---|---|
| 1 | Welcome to [Telco] | template | UTILITY | Number activated |
| 2 | Plan Usage Update | template | UTILITY | Mid-cycle data threshold (e.g. 80% used) |
| 3 | Top-up Reminder | template | UTILITY | Balance below threshold |
| 4 | Customer Taps "Top Up" | customer_reply | REPLY | → opens payment Flow |
| 5 | Top-up Confirmation | template | UTILITY | Payment processed |
| 6 | Roaming Pack Offer | template | MARKETING | Geo-fence trigger (departure airport) |
| 7 | International Roaming Activated | template | UTILITY | Roaming pack provisioned |
| 8 | Speak to Agent | template | SERVICE | Customer escalation |

---

## Healthcare — Appointments + Reminders

Tread carefully — health-related content has stricter WABA template approval rules in many markets.

| # | Title | Type | Category | Trigger |
|---|---|---|---|---|
| 1 | Appointment Booking Confirmation | template | UTILITY | EMR appointment created |
| 2 | 24-hour Reminder | template | UTILITY | Scheduled job, T-24h |
| 3 | Customer Confirms / Reschedules | customer_reply | REPLY | Tap "Confirm" or "Reschedule" |
| 4 | Reschedule Flow | template | UTILITY | → opens calendar Flow |
| 5 | Pre-Visit Forms Reminder | template | UTILITY | Patient profile incomplete |
| 6 | Check-in QR Code | template | UTILITY | T-1h, sends location + QR |
| 7 | Post-Visit Care Summary | template | UTILITY | Visit closed in EMR |
| 8 | Lab Results Available | template | UTILITY | Results signed by clinician (link out to portal) |

---

## Logistics — Delivery + Reschedule

Strong fit for last-mile and B2C shipping.

| # | Title | Type | Category | Trigger |
|---|---|---|---|---|
| 1 | Shipment Created | template | UTILITY | Carrier label generated |
| 2 | Out for Delivery | template | UTILITY | Driver scan at depot |
| 3 | Driver Approaching (15 min) | template | UTILITY | GPS proximity trigger |
| 4 | Customer Replies "Not Home" | customer_reply | REPLY | → opens reschedule Flow |
| 5 | Reschedule Confirmed | template | UTILITY | Updated delivery slot |
| 6 | Delivered with Photo Proof | template | UTILITY | POD captured |
| 7 | Rate the Driver | template | MARKETING | Post-delivery NPS |

---

## Choosing between verticals

If the user gives you a customer name but not a use case, default to:
- **Banks** → Credit Card Acquisition (or CRM Bulk Reminders if the contact is in collections / ops)
- **Insurers** → Policy Renewal
- **Retailers** → Order Lifecycle
- **Telcos** → Onboarding + Plan Management
- **Hospitals/clinics** → Appointments
- **Couriers / 3PLs** → Delivery + Reschedule

Confirm the choice with the user before scripting — you can save a round-trip if the assumption is wrong.

## Composing across verticals

Real demos sometimes need two journeys back-to-back (acquisition + servicing). Keep them in the same `DEMO_DATA[market].steps` array and add a `system_event` divider step in between:

```javascript
{
  id: 10, title: "— 30 days later —", type: "system_event",
  category: "SYSTEM", trigger: "Time-based event",
  preview: "30 days have passed. Customer is now an active cardholder."
}
```

This gives the audience a clear narrative break.
