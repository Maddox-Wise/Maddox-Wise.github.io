# Network Monitoring Alert Workflow

**Date:** 2026-06-02  
**Project Area:** Incident preparation and network monitoring  
**Objective:** Improve visibility into active devices on the local network and create a lightweight alerting workflow for unknown connections.  

---

## Overview

This project documents a lightweight network monitoring workflow for identifying unknown active devices on a local network. The workflow is designed for homelab and small-network environments where quick visibility into unexpected connections is useful for incident preparation and response.

---

## Changes Made

- Documented a Python-based monitoring workflow for detecting active devices on the LAN
- Used an approved-device allowlist to separate known systems from unknown connections
- Included optional Tailscale peer awareness for environments that use both local and overlay networking
- Added Discord webhook alerting for newly detected unknown active devices
- Kept alert behavior focused on actionable events instead of repeated noise
- Connected the workflow to the existing Unknown Device Monitor project

---

## Validation

- Reviewed the monitoring flow for local scanning, allowlist comparison, optional overlay-network checks, and alert delivery
- Confirmed public notes avoid exposing webhook URLs, internal-only addresses, private hostnames, or device-specific secrets
- Kept setup details reproducible with placeholder values where sensitive information would normally appear

---

## Follow-Up

- Add example screenshots or sanitized alert output if useful for future review
- Expand the allowlist as trusted devices are added to the network
- Consider logging unknown-device history locally for incident review
- Review scheduled execution timing to balance visibility with unnecessary scans
