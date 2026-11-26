# SOC / SIEM Lab: Investigating Suspicious Activity on My Own Server

I loaded the logs from my own AWS EC2 server into Splunk, searched them for suspicious activity, investigated one event and wrote an incident report.

**Author:** Varshini Murali, cybersecurity student at Murdoch University

> Fill in every [bracket] with your own results. Only write what your logs really show. Delete this note when you're done.

## Problem

A server on the internet is scanned and attacked all the time. I wanted to learn how a SOC analyst finds that activity in logs, so I used the logs from my own ICT171 cloud server and investigated them.

## Tools

| Tool | What I used it for |
| --- | --- |
| AWS EC2 (Ubuntu) with Apache | The server that produced the logs |
| Splunk Enterprise | The SIEM I loaded the logs into and searched |
| `auth.log` | Login attempts to the server |
| `access.log` | Requests made to the website |
| MITRE ATT&CK | Naming the techniques I found |

## Action

1.Created a safe lab using my Ubuntu server and Splunk Enterprise.
 2.Generated approximately 6 failed SSH login attempts using the fake username fakeadmin, and created multiple requests to pages that do not exist (/admin and /wp-login.php) between approximately 13:19 and 13:25 on 23 November 2026.
3.Copied auth.log and access.log from the server to my laptop.
4.Loaded both logs into Splunk using the source types linux_secure and access_combined.
5.Ran the searches in searches.md.
6.Investigated the most active source IP address.
7.Created an alert for repeated failed login attempts.
8.Wrote an incident report based on the findings.

## Findings

| What I looked for | What I found |
| --- | --- |
| Total failed login attempts | [23] |
| Top IP by failed logins | 172.23.112.xxx  with [23] attempts |
| Most tried usernames | [fakeadmin] |
| Time of the attempts | Approximately 13:19–13:21 on 23 November 2026 |
| Pages scanned | /admin (9 requests), /wp-login.php (7 requests), /favicon.ico (2 requests) |
| Any successful login from the suspicious IP? | [No. I checked using sourcetype=linux_secure "Accepted" and found no successful logins from 172.23.112.1.] |



### Suspicious activity

| Activity | Meaning | MITRE ATT&CK |
| --- | --- | --- |
| Many failed logins from one IP | Brute force attempt | T1110 Brute Force |
| Requests for pages that don't exist | Scanning for weak spots | T1595 Active Scanning |

### Investigation of one IP

- **Timeline:** At approximately 13:19 on 23 November 2026, IP address 172.23.112.xxx began making SSH authentication attempts using the username fakeadmin. Multiple invalid-user, failed-password and authentication-failure events were recorded until approximately 13:21.

- **Targets:** SSH service
- **Result:** The activity was consistent with a brute-force login attempt. No successful login was observed from this IP address and there was no evidence of compromise.

## Risk

Low, because repeated failed login attempts and web scanning activity were detected, but no successful authentication was observed from the suspicious IP address.

## Alert

I created an alert named "SSH brute force". It runs when one IP has more than 5 failed logins in 10 minutes.



## Recommendations

1. Limit SSH (port 22) to my own IP address in the security group.
2. Keep password login for SSH switched off.
3. Install fail2ban to block IPs that keep failing.
4. Keep the server updated.
5. Send logs to a SIEM all the time, not just for this lab.

## Lessons learned

- Splunk can be used to quickly identify suspicious activity in authentication and web server logs.
- Repeated failed SSH login attempts from a single IP address can indicate brute-force attack activity.
- Web server logs can reveal reconnaissance activity through requests for common pages such as /admin and /wp-login.php.

## Files

- `searches.md`: the Splunk searches I used
- `report.pdf`: the full incident report
- `screenshots/`: screenshots of my results
