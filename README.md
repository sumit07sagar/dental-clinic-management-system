# Clinic CRM — a dental practice management system on Google Apps Script

A complete patient, appointment and finance system for a dental practice, running
entirely inside the clinic's own Google account. No servers, no monthly hosting bill,
and no third party holding the patient records.

Built for a real clinic, used daily, then productized so it can be stood up for a new
practice with one command.

> **This is a showcase repository.** It documents what the system does and how it is
> built. The deployable source is private and licensed per practice —
> see [Working with me](#working-with-me).

---

## The problem it solves

Small dental practices are stuck between paper and enterprise software. Practice
management suites cost hundreds a month, assume a receptionist, and put patient
records on someone else's servers. Spreadsheets are free but nobody can use them from
the chair, and the money never adds up at month end.

This sits in between: the clinic keeps its data in its own Google account, staff use
it from a phone, and it costs nothing to run.

---

## What it does

### For the doctor
- Record a visit in about thirty seconds, from the chair
- Every patient's full treatment history as a timeline
- **Multi-visit treatment plans with a running balance** — an RCT or crown course
  spans weeks and several payments; the system tracks the course, not just the visits
- Month-by-month P&L: billed, collected, lab costs, expenses, net
- Outstanding balances, quotes that never converted, patients due for recall
- Visits where billing was forgotten, surfaced with a one-tap fix

### For the assistant
- Register a patient and book them in
- Book follow-ups against the clinic's real Google Calendar
- Upload X-rays and photos straight to the clinic's Drive
- **Never sees a single financial figure**

### Built in
- WhatsApp confirmation to the patient, in the clinic's own name
- Duplicate-patient detection that catches typos — "Surbhi" vs "Surabhi"
- A guard against the same payment being recorded twice on one day

---

## Screenshots

### First-run setup

![Setup wizard](screenshots/01-setup-wizard.jpg)

The screen a new clinic sees before anything else. It collects everything the app
needs — clinic name, brand colour, staff names, PINs, calendar, opening hours,
timezone, currency — and **refuses to complete while any shipped placeholder is still
in place.**

That check exists because of a real incident. An earlier build made the clinic name
configurable but nobody filled the cell in, so the moment WhatsApp confirmations
started reading from config, patients would have received *"Your appointment at Clinic
Galla is confirmed."* Making a value configurable is only half the job; the other half
is refusing to run until it is set.

---

## How it is built

**Google Apps Script + one Google Sheet.** Two files: a server (`Code.gs`) and a
single-file client (`App.html`). No build step, no dependencies, no framework. What is
in the repository is exactly what runs.

**Every clinic gets its own deployment.** Deliberately not multi-tenant — separate
Sheet, separate script, separate patients, separate money. A clinic's data cannot leak
into another's because they never share a database.

**Nothing clinic-specific lives in the code.** Clinic name, PINs, staff names,
calendar, hours, booking link, Drive folder, brand colour, timezone and currency are
all rows in a config tab. The code is byte-identical across every deployment, which is
what makes a single bug fix pushable to all of them.

**The spreadsheet builds itself.** On first run the script creates its own sheets and
column headers, so a new clinic is stood up from source alone — there is no template
spreadsheet to copy, and therefore no route by which one clinic's records could be
copied into another's.

### Some decisions worth explaining

**Role separation is enforced on the server, not in the interface.** When an assistant
signs in, the payload sent to the browser simply does not contain money — no totals,
no dues, no history. Hiding a column in the UI is not a permission model; the data
never leaves the server.

**A shared code library was considered and rejected.** It would have let one change
reach every clinic instantly, but Apps Script requires each caller to have viewer
access on the library — and viewer means readable source. It also removes any way to
stage a rollout, so one bad save breaks every clinic at once. Per-clinic deployment
with a scripted fleet push was the better trade.

**Native dialogs turned out to be suppressed.** The duplicate-patient warning used
`confirm()`, which silently returns false inside the Apps Script iframe — so the
"merge with existing patient" branch had been unreachable in production, quietly
creating the duplicates it was written to prevent. Replaced with an inline panel.
Every guard in the app now uses DOM prompts.

---

## Testing

A read-only regression suite runs against a clinic's live data and writes nothing, so
it is safe during clinic hours. It has two kinds of check:

- **Pure** — fixed inputs and outputs for the helpers (name normalisation, fuzzy
  matching, date formatting, currency).
- **Invariant** — facts that must hold for *any* correct data: every patient's paid
  total equals the sum of their rows, every treatment-plan balance equals billed minus
  paid, monthly net equals collected minus costs, and **the assistant payload contains
  no financial key.**

Invariant tests need no fixtures and get stronger as a clinic's data grows. 73 checks
on a fresh install.

---

## Security

- PINs are compared server-side and never sent to the browser
- Eight wrong PINs in fifteen minutes freezes logins for ten minutes, with a
  deliberate delay on every wrong attempt
- The app runs as the deploying account, so clinic staff never need Google accounts or
  access to the spreadsheet
- Known limitations are documented honestly for buyers rather than glossed over

---

## Working with me

I build small, durable business systems — the kind a two-person clinic or workshop
actually uses every day, rather than software that impresses in a demo and gets
abandoned in a month.

**The deployable source is private.** I'm happy to walk through it on a call, or grant
read access to a serious prospect.

- Deployment for a practice, including setup and handover
- Custom builds on the same foundation for other single-location businesses
- Google Workspace automation generally

Get in touch through the profile that brought you here.

---

*Copyright (c) 2026 Growth 100x. This showcase repository is documentation. The
software it describes is licensed per practice and is not open source.*
