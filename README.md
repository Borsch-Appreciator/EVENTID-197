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
Starting off this case, in the rule it lists possibilty of CVE-2023-22515 which is described at cve.org as (Score: 10.0	Severity:CRITICAL	3.0	Vector stirngCVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H) Breaking down the CVSS the attack vector (AV) is network, Attack complexity (AC) Low, Privilage requeired(PR): None, User Interaction (UI) No User Action, Scope (S): Changed, Impact (I): High, Availility (A): High. All in all if this CVE is being exploited this is definitely a high priority issue that should be escalated quickly. First thing I want to check it is to see if the source address has attempted to attack only this end point or if it has targets multiple devices on the network. Plugging in the source address returns 3 events, all of which are targeting the same destination address which is the one we have on the alert information

## Initial Investigation

Network Events:

|field|value|
|-----|-----|
source address | 43.130.1.222
source port|42312
destination address|172.16.17.234
destination_port| 80
time Nov, 09, 2023, 09:47 AM
Value
Proxy











Raw Log
Request URL: /server-info.action?bootstrapStatusProvider.applicationConfig.setupComplete=false
Request Method

GET
Response Code

200
User-Agent

curl/7.88.1


Field
type

source_address

source_port

destination_address

destination_port

time

Value
Proxy

43.130.1.222

4323

172.16.17.234

80

Nov, 09, 2023, 09:47 AM

Raw Log
Request URL

/setup/setupadministrator.action
Request Method

POST
Response Code

302
User-Agent

curl/7.88.1


Field
type

source_address

source_port

destination_address

destination_port

time

Value
Proxy

43.130.1.222

45321

172.16.17.234

80

Nov, 09, 2023, 09:48 AM

Raw Log
Request URL

/setup/finishsetup.action
Request Method

POST
Response Code

200
User-Agent

curl/7.88.1


It looks like the commands /server-info.action?bootstrapStatusProvider.applicationConfig.setupComplete=false, /setup/setupadministrator.action
Request Method, and  /setup/setupadministrator.action all posted without issue which is alarming. With these findings I'd escalate and recommend immediate quaranting of the affected serv. Switching to the EndPoint Security










