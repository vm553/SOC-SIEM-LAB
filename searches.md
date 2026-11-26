# Splunk searches

These are the searches I ran on my own server's logs. Field names can differ between Splunk versions, so I checked the fields list on the left side of the search page.

## 1. Which IP addresses failed to log in the most?

```
sourcetype=linux_secure ("Failed password" OR "Invalid user") | rex "from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})" | stats count by src_ip | sort - count
```

**Why:** many failed logins from one IP in a short time is a sign of a brute force attempt.

## 2. Which usernames were tried?

```
sourcetype=linux_secure "Invalid user" | rex "Invalid user (?<user>\S+)" | stats count by user | sort - count
```

**Why:** attackers guess common names like root, admin and test.

## 3. When did the failures happen?

```
sourcetype=linux_secure ("Failed password" OR "Invalid user") | timechart span=1h count
```

**Why:** a sudden spike shows when an attack started and stopped.

## 4. Which pages returned a 404 error, and who asked for them?

```
sourcetype=access_combined status=404 | stats count by clientip, uri_path | sort - count
```

**Why:** repeated requests for pages that don't exist, like /admin or /wp-login.php, are a sign of scanning.

## 5. Did any login succeed?

```
sourcetype=linux_secure "Accepted"
```

**Why:** this tells me whether an attacker got in, or whether the only successful logins were mine.

## 6. Investigating one IP (replace the IP address)

```
sourcetype=linux_secure "[IP ADDRESS]" | sort _time
```

**Why:** this shows everything one IP did, in time order, so I can tell the story of what happened.

## 7. Alert search: more than 5 failed logins from one IP

```
sourcetype=linux_secure ("Failed password" OR "Invalid user") | rex "from (?<src_ip>\d{1,3}(?:\.\d{1,3}){3})" | stats count by src_ip | where count > 5
```

Saved as an alert named "SSH brute force".
