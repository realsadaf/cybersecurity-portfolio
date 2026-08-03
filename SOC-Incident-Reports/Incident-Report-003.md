# Incident Report – Backdoored Browser Extension: C2 Beacon & Session Cookie Theft

**Severity:** High  
**Category:** Browser Compromise / Credential Access  
**MITRE ATT&CK:** T1539 – Steal Web Session Cookie

---

## Incident Summary

Investigated a compromised Chrome browser extension that was trojanized through a malicious update. The extension established communication with an attacker-controlled Command and Control (C2) server, retrieved a remote configuration file, harvested authenticated session cookies and an API token from the victim's browser, and exfiltrated the stolen data to a VULTR-hosted server. Proxy, firewall and SIEM logs were analysed to reconstruct the attack.

---

## Investigation

### Step 1 – Identified Initial C2 Communication

- Reviewed proxy logs to identify outbound HTTPS requests.
- Located the request where the extension retrieved a remote JSON configuration file referenced by a browser local storage key.
- Confirmed this as the initial C2 beacon.

### Step 2 – Traced Data Exfiltration

- Analysed outbound POST requests.
- Identified the destination receiving the highest volume of transmitted packets.
- Confirmed the attacker-controlled subdomain used for cookie and API token exfiltration.

### Step 3 – Identified the Malicious Script

- Filtered browser activity for **background.js**.
- Retrieved and documented the script's SHA-256 hash.
- Recorded the hash as an Indicator of Compromise (IOC).

### Step 4 – Examined Browser Storage

- Queried SIEM logs to identify the browser local storage key used by the extension.
- Confirmed the downloaded configuration was stored locally before execution.

### Step 5 – Assessed Attacker Objective

- Determined the extension harvested authenticated session cookies and an API token to hijack active user sessions without requiring passwords.
- Mapped the activity to **MITRE ATT&CK T1539 – Steal Web Session Cookie (Credential Access).**

---

## Indicators of Compromise (IOCs)

- Malicious C2 domain
- Attacker-controlled exfiltration subdomain
- `background.js`
- SHA-256 hash of malicious script
- Browser local storage key

---

## Recommendations

- Remove the malicious browser extension from affected systems.
- Invalidate compromised session cookies and API tokens.
- Reset affected user credentials if compromise is confirmed.
- Block identified C2 domains and IP addresses at the firewall/proxy.
- Implement browser extension allow-listing to reduce future risk.

---

## Outcome

Successfully reconstructed the attack chain from initial C2 beaconing through credential theft and data exfiltration using proxy, firewall and SIEM logs. Identified attacker infrastructure, malicious scripts, Indicators of Compromise (IOCs), and mapped the attack to the MITRE ATT&CK framework.
