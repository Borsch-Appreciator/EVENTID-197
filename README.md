# EVENTID-197

|field|Value|
|-----|-----|
EventID | 197|
Event Time |Nov, 09, 2023, 09:47 AM
Rule |SOC235 - Atlassian Confluence Broken Access Control 0-Day CVE-2023-22515
Level |Security Analyst
Hostname |Confluence Data Center
Destination IP Address |172.16.17.234
Source IP Address |43.130.1.222
HTTP Request Method |GET
Requested URL |/server-info.action?bootstrapStatusProvider.applicationConfig.setupComplete=false
User-Agent |curl/7.88.1
Alert Trigger Reason |This activity may be indicative of an attempt to exploit the CVE-2023-22515 vulnerability, which could potentially lead to create a new administrator user.
Device Action |Allowed


## Initial alert first look
When beginning this investigation, I immediately noticed the alert referenced CVE-2023-22515.
According to CVE.org, this vulnerability has a CVSS score of 10.0 (Critical) with the following vector:
Plain Text1CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:HShow more lines
Breaking down the CVSS score:

MetricValueAttack Vector (AV)NetworkAttack Complexity (AC)LowPrivileges Required (PR)NoneUser Interaction (UI)NoneScope (S)ChangedConfidentiality (C)HighIntegrity (I)HighAvailability (A)High
Overall, if this vulnerability is actively being exploited, it would be considered a high-priority incident that should be escalated quickly.
The first thing I wanted to determine was whether the source IP had targeted only this endpoint or if it had attempted communication with multiple systems across the environment.
Searching the logs for the source address 43[.]130[.]1[.]222 returned three events, all targeting the same host:
172[.]16[.]17[.]234 (Confluence Data Center)
This suggests the activity was specifically focused on this server rather than broad network scanning.

## Initial Investigation


### Network Event #1

| Field | Value |
|-------|-------|
| Source Address | 43[.]130[.]1[.]222 |
| Source Port | 42312 |
| Destination Address | 172[.]16[.]17[.]234 |
| Destination Port | 80 |
| Time | Nov 09, 2023, 09:47 AM |
| Value | Proxy |
| Request URL | /server-info.action?bootstrapStatusProvider.applicationConfig.setupComplete=false |
| Request Method | GET |
| Response Code | 200 |
| User-Agent | curl/7.88.1 |

### Network Event #2

| Field | Value |
|-------|-------|
| Source Address | 43[.]130[.]1[.]222 |
| Source Port | 4323 |
| Destination Address | 172[.]16[.]17[.]234 |
| Destination Port | 80 |
| Time | Nov 09, 2023, 09:47 AM |
| Value | Proxy |
| Request URL | /setup/setupadministrator.action |
| Request Method | POST |
| Response Code | 302 |
| User-Agent | curl/7.88.1 |

### Network Event #3

| Field | Value |
|-------|-------|
| Source Address | 43[.]130[.]1[.]222 |
| Source Port | 45321 |
| Destination Address | 172[.]16[.]17[.]234 |
| Destination Port | 80 |
| Time | Nov 09, 2023, 09:48 AM |
| Value | Proxy |
| Request URL | /setup/finishsetup.action |
| Request Method | POST |
| Response Code | 200 |
| User-Agent | curl/7.88.1 |

## Analysis

The sequence of requests immediately stood out:

server-info.action?bootstrapStatusProvider.applicationConfig.setupComplete=false
setup/setupadministrator.action
setup/finishsetup.action

These URLs closely align with publicly documented exploitation activity associated with CVE-2023-22515.
What is particularly concerning is that the requests completed successfully:

RequestResponseserver-info.action200setupadministrator.action302finishsetup.action200
The successful HTTP response codes indicate that the requests were processed by the Confluence server rather than blocked outright.
Additionally, the use of:
curl/7.88.1

as the User-Agent is suspicious. Legitimate users typically access Confluence through a web browser, whereas attackers frequently use command-line tools such as curl to automate exploitation attempts.
Based on the evidence collected so far:

The source IP is directly targeting the vulnerable Confluence server.
The requests match known exploitation paths associated with CVE-2023-22515.
The requests were accepted and processed successfully.
The User-Agent suggests automated activity rather than a normal user interaction.

At this point, I would consider the likelihood of compromise to be extremely high.  Given the severity of the vulnerability and the evidence observed during the investigation, I would immediately escalate the incident and recommend quarantining the affected Confluence server to prevent any additional malicious activity.

## Recommended actions:

Isolate the affected Confluence host from the network.
Review authentication logs for unauthorized administrator account creation.
Examine audit logs for configuration changes.
Search for additional indicators of compromise tied to the source IP.
Verify whether the vulnerable Confluence version is installed.
Apply vendor-recommended remediation and security updates.
Perform a full scope assessment to determine whether any persistence mechanisms were established.

This alert appears to be a true positive and a likely attempt to exploit CVE-2023-22515, a critical Atlassian Confluence vulnerability that can allow unauthorized administrator account creation.
The attack originated from 43[.]130[.]1[.]222 and targeted a single Confluence Data Center server. The observed requests closely match known exploitation behavior, and the successful HTTP response codes suggest the requests were processed by the application.
Based on the available evidence, I would treat this as a high-priority security incident and initiate containment and escalation procedures immediately.
Provide your feedback on BizChat


It looks like the commands /server-info.action?bootstrapStatusProvider.applicationConfig.setupComplete=false, /setup/setupadministrator.action
Request Method, and  /setup/setupadministrator.action all posted without issue which is alarming. With these findings I'd escalate and recommend immediate quaranting of the affected serv. Switching to the EndPoint Security










