---
title: Improving Security Data Through OCSF Normalization
description: A vendor-neutral schema for security telemetry — GuidePoint Security Capstone, Summer 2026
---

# Improving Security Data Through OCSF Normalization

**A Vendor-Neutral Schema for Security Telemetry**  
GuidePoint Security Capstone Project · Summer 2026

**Maddox Wise** — Security Operations Intern, GuidePoint Security  
Studying Cybersecurity, University of Tulsa · Formerly Student Help Desk Technician · Mentor: Tyler Irons

---

## Why: every product describes the same event in its own words

Two vendors, one allowed outbound session, 4,812 bytes:

| Source | Raw event |
|---|---|
| Palo Alto PAN-OS 11.1 | `TRAFFIC,end,2026/07/21 20:00:00 src=10.10.20.15:51514 dst=93.184.216.34:443 app=ssl action=allow bytes=4812` |
| Cisco ASA 9.x | `%ASA-6-302014: Teardown TCP connection 101001 for outside:93.184.216.34/443 to inside:10.0.0.15/51514 bytes 4812` |

Different words, identical facts. So why can't one query read both?

---

## What is OCSF? One schema, any vendor

**Log** (any product, any format) → **Pipeline** (parse, map, validate) → **Output** (one shape, common fields)

Only the middle stage changes per source; the output shape stays the same — that is the whole point.

> OCSF defines the fields — it does not fill them. Every source needs a mapping written, tested, and correlated.

### One language, every source

- Firewall, cloud, endpoint — same question, same field name
- Analysts learn the schema once, not a dialect per product
- New hires are useful on day one, not week six

### Swap vendors, keep everything

- Rip out a product and only its mapping changes
- Queries, dashboards, and detections still point at the same fields
- Switching a tool costs one mapping rewrite, not a rebuild of every downstream query

### Correlate without duct tape

- One event shape means joins work with no lookup tables
- Network, endpoint, and identity events line up on shared keys
- Coverage gaps become obvious instead of hiding in formats

---

## How: architecture — raw log → Cribl Stream → OCSF event

Raw firewall logs (PAN-OS & ASA) through the `OCSF-ASA-Cribl` pipeline:

1. Parse the syslog envelope
2. Filter to in-scope message IDs
3. Set OCSF base fields and keep raw
4. Parse message body per event type
5. Derive outcome, action, and status
6. Assemble objects, strip scratch fields

Output: **OCSF 1.8.0 event JSON**. Built with stock Cribl functions only — no custom development. The full raw event is retained in `raw_data` on every event that maps.

---

## One log, transformed

**Before — raw Cisco ASA syslog**

```
%ASA-4-106023: Deny tcp src outside:203.0.113.25/4444 dst inside:10.0.0.10/22 by access-group "outside_access_in" [0x0, 0x0]
```

To read this you already have to know that 106023 means denied by an access list, that severity 4 is a warning rather than a rank, and that interface names — not fields — imply direction.

**After — OCSF Network Activity (class 4001)**

```json
{
  "class_uid": 4001,
  "category_uid": 4,
  "severity_id": 3,
  "action": "Denied",
  "action_id": 2,
  "disposition": "Blocked",
  "disposition_id": 2,
  "status": "Failure",
  "status_id": 2,
  "is_alert": true,
  "src_endpoint": "203.0.113.25:4444",
  "dst_endpoint": "10.0.0.10:22",
  "firewall_rule": "outside_access_in",
  "metadata.product": "Cisco ASA",
  "time": 1784140438280,
  "message": "ASA denied TCP 203.0.113.25:4444 -> 10.0.0.10:22 [outside_access_in]",
  "raw_data": "<original event>"
}
```

---

## A baseline, not a verdict

Normalization is interpretive, not mechanical. This build takes the conservative reading and keeps every decision visible and editable.

- Every source has judgment calls; vendor terms rarely mean what OCSF means
- Ambiguous fields stay unmapped rather than forced
- Enrichment is never inferred from free text
- Every binding is a GUI setting — no rebuild required
- Vendor extras wait in an unmapped namespace
- `raw_data` keeps the full event for re-mapping

---

## Who benefits from common field names

- **SOC analyst** — `src_endpoint.ip` is `src_endpoint.ip` no matter which product emitted the event
- **Detection engineer** — rules reference the schema, so `class_uid` 4001 and `activity_id` survive a change of tooling
- **Platform engineer** — map each source once into one schema; still real work, but built once, not per downstream tool
- **Security leadership** — one vocabulary across the estate; analytics move between SIEMs without rewriting field names

---

## Who keeps the mapping alive

- **Platform engineering** — owns the pipeline; stock Cribl functions mean the next owner edits it in the interface
- **Detection engineering** — signs off that vendor terms map to matching OCSF concepts, not just similar names
- **Source owners** — a firmware upgrade or new message ID is a mapping change, not silent data-quality drift
- **Schema stewardship** — each OCSF release is scheduled work: re-validate the mapping rather than assume it holds

---

OCSF · GuidePoint Security Capstone · Summer 2026 · Copyright ©2026 · guidepointsecurity.com
