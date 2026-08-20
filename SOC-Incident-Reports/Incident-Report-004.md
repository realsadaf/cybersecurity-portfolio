# Incident Report – Trusted Domain, Untrusted Destination: Open-Redirect Phishing

**Severity:** High  
**Category:** Phishing / Credential Theft / Open Redirect  
**MITRE ATT&CK:** T1566.002 – Phishing: Spearphishing Link  
**Additional Techniques:** T1204.001 – User Execution: Malicious Link | T1071.001 – Web Protocols

---

## Incident Summary

Investigated a phishing campaign targeting a finance clerk at **Larkfield Mutual Assurance**.

The attacker used an **open redirect on a trusted financial domain** to bypass URL reputation filtering. The phishing email directed the victim to a trusted `citi.com` URL, which returned an HTTP 302 redirect to an attacker-controlled intermediate server.

The intermediate server then redirected the victim to a fake Microsoft 365 login page, where the victim submitted their corporate credentials.

A subsequent successful Entra ID sign-in from an unfamiliar IP address with `mfa=not-required` and `session_reused=true` provided evidence of potential credential or session replay.

---

## Attack Chain

```text
Phishing Email
      ↓
Trusted citi.com URL
      ↓
HTTP 302 Open Redirect
      ↓
frudyj.codesandbox.io
      ↓
JavaScript Redirect
      ↓
Fake Microsoft 365 Login
      ↓
gw8aes.office-docs.net
      ↓
Credentials Submitted
      ↓
Suspicious Entra ID Sign-in
      ↓
37.120.234.53
```

---

## Investigation

### 1. Identify the Trusted Domain

The mail gateway recorded an inbound phishing email sent to:

`marcella.deniston@larkfield-mutual.com`

The message used an **"Undelivered Mails" / "Release My Messages"** lure and impersonated Microsoft Exchange.

The embedded URL used:

`accountonline.citi.com`

The gateway classified the URL as:

`trusted-brand (allow-listed)`

**Finding:** The attacker abused the trusted reputation of `citi.com` to increase the likelihood that the phishing link would bypass URL filtering.

---

### 2. Follow the Open Redirect

The web proxy recorded the victim clicking the trusted URL.

The request returned:

`HTTP 302`

The `Location` header redirected the browser to:

`https://frudyj.codesandbox.io/release`

**Evidence:**

- Requested domain: `citi.com`
- Status code: `302`
- Redirect destination: `frudyj.codesandbox.io`
- User: `marcella.deniston`
- Host: `lkf-fin-wks-217`

**Finding:** The trusted `citi.com` URL acted as the first-stage redirector, sending the victim to an external destination.

---

### 3. Identify the Intermediate Server

The intermediate hostname identified through SIEM and proxy logs was:

`frudyj.codesandbox.io`

This server acted as a **redirect/cushion layer** between the trusted domain and the final credential-harvesting page.

The attack chain became:

`citi.com → frudyj.codesandbox.io → credential-harvesting site`

---

### 4. Identify the Credential-Harvesting Host

Further proxy activity showed the victim reaching:

`gw8aes.office-docs.net`

The following request was identified:

`POST /auth/login`

The request contained:

- `login`
- `passwd`
- `flowToken`

The submitted username was:

`marcella.deniston@larkfield-mutual.com`

### Evidence Assessment

| Evidence | Assessment |
|---|---|
| POST request | Suspicious – data was submitted to the server |
| `login`, `passwd`, `flowToken` | Strong indicator of credential submission |
| `gw8aes.office-docs.net` | Suspicious external authentication host |
| `/auth/login` | Strong indicator of a login endpoint |
| Corporate username submitted | Supporting evidence |
| HTTP 200 | Not malicious by itself |

**Finding:** The evidence indicates that the victim submitted corporate credentials to an attacker-controlled authentication endpoint.

---

### 5. Identify Potential Credential Replay

Two successful Entra ID sign-ins were observed around the incident.

#### Legitimate Sign-in

```text
2025-11-04 13:02:00

User: marcella.deniston@larkfield-mutual.com
Application: Office 365 Exchange Online
Source IP: 198.51.100.61
Result: Success
MFA: Satisfied
```

#### Suspicious Sign-in

```text
2025-11-04 13:52:00

User: marcella.deniston@larkfield-mutual.com
Application: Office 365 Exchange Online
Source IP: 37.120.234.53
Result: Success
MFA: Not required
Session reused: True
```

**Finding:** The second sign-in originated from a different IP address and showed `mfa=not-required` and `session_reused=true`.

This is consistent with potential **credential or session replay** following the phishing event.

> `mfa=not-required` does not by itself prove that the attacker bypassed MFA. Additional authentication telemetry would be required to determine why MFA was not required.

The suspicious source IP identified was:

`37.120.234.53`

---

## MITRE ATT&CK Mapping

### T1566.002 – Phishing: Spearphishing Link

The attacker delivered a phishing email containing a malicious link designed to redirect the victim to a credential-harvesting page.

### T1204.001 – User Execution: Malicious Link

The attack required the victim to interact with the **"Release My Messages"** link.

### T1071.001 – Application Layer Protocol: Web Protocols

HTTPS/HTTP communications were used throughout the redirect and credential-harvesting chain.

### Credential Access

The attacker obtained the victim's credentials through a spoofed authentication page.

---

## Indicators of Compromise (IOCs)

| Type | Indicator | Description |
|---|---|---|
| Domain | `accountonline.citi.com` | Trusted domain abused for open redirect |
| Hostname | `frudyj.codesandbox.io` | Intermediate redirect server |
| Hostname | `gw8aes.office-docs.net` | Credential-harvesting host |
| IP | `37.120.234.53` | Suspicious Entra ID source IP |
| Username | `marcella.deniston@larkfield-mutual.com` | Potentially compromised account |
| URI | `/auth/login` | Credential submission endpoint |

---

## Recommended Response

1. Reset the affected user's credentials.
2. Revoke active sessions and authentication tokens.
3. Investigate the suspicious Entra ID activity from `37.120.234.53`.
4. Block the identified malicious hostname at the proxy/DNS layer.
5. Search the environment for other users who received the same phishing email.
6. Search proxy logs for additional requests to the identified infrastructure.
7. Review the user's account for suspicious activity following the credential submission.
8. Add detection rules for suspicious external authentication domains and open-redirect chains.
9. Educate users about phishing links that use trusted domains as redirectors.

---

## Outcome

Successfully reconstructed the phishing attack from initial email delivery through credential theft and potential account/session replay.

The investigation identified:

- The trusted domain abused by the attacker
- The open-redirect mechanism
- The intermediate redirect server
- The credential-harvesting host
- The potentially compromised user account
- The suspicious Entra ID source IP
- Relevant MITRE ATT&CK techniques

The investigation demonstrated the use of **mail gateway, web proxy, SIEM and Entra ID telemetry** to correlate events across multiple stages of a phishing attack.
