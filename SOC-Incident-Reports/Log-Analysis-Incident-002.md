simulated lab logs

Time: 2025-02-05 10:12:01
Event ID: 4625
Account Name: admin
Source IP: 185.234.219.12
Logon Type: 3
Failure Reason: Bad password

Time: 2025-02-05 10:12:09
Event ID: 4625
Account Name: admin
Source IP: 185.234.219.12
Logon Type: 3
Failure Reason: Bad password

Time: 2025-02-05 10:12:31
Event ID: 4624
Account Name: admin
Source IP: 185.234.219.12
Logon Type: 3

# SOC Incident Report – Log Analysis (Brute Force Detection)

## Incident Summary
Multiple failed login attempts were detected from a single IP address followed by a successful login, indicating a possible brute-force attack.

## Detection Method
Security log monitoring of Windows authentication events.

## Log Evidence
- Event ID 4625 – Failed logon (multiple occurrences)
- Event ID 4624 – Successful logon

## Indicators of Compromise (IOCs)
- Source IP: 185.234.219.12
- Target Account: admin

## Attack Stage
Credential Access

## MITRE ATT&CK Mapping
- T1110 – Brute Force
- T1110.001 – Password Guessing

## Impact Assessment
Compromise of administrative credentials could lead to full system access.

## Containment & Remediation
- Block source IP
- Force password reset on affected account
- Enable account lockout policy
- Review additional authentication logs

## Detection Improvement
Implement alerts for excessive failed login attempts followed by successful authentication.

## Lessons Learned
Repeated authentication failures followed by success is a strong indicator of brute-force activity.


