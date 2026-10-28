# Home Lab: Splunk SIEM with Metasploitable 2

A small VirtualBox lab where a vulnerable Linux machine sends its logs to Splunk, and I attack it to see what the SIEM actually catches. The first attack didn't show up at all, which turned out to be the most useful part.

**Stack:** VirtualBox, Ubuntu Server 20.04, Splunk Enterprise, Metasploitable 2 (Apache, DVWA), rsyslog

![Lab diagram](screenshots/lab-diagram.png)

## Setup

| Machine | OS | Role | Network |
|---|---|---|---|
| Splunk VM | Ubuntu Server 20.04 | Runs Splunk Enterprise | NAT first, then Host-Only |
| Metasploitable VM | Metasploitable 2 (.vmdk) | Vulnerable target (Apache, DVWA), forwards logs | Host-Only |
| Host laptop | - | Browser for DVWA and Splunk (192.168.56.1) | Host-Only |

**Metasploitable VM:** created a Linux VM in VirtualBox and attached the Metasploitable 2 disk image.

**Splunk VM:** created a second VM from the Ubuntu Server 20.04 ISO. It started on a NAT adapter so I could download Splunk with `wget`. After the download I switched it to Host-Only, so the lab is only reachable from my laptop and the other VM, not the internet.

![VirtualBox network settings](screenshots/network-settings.png)

## Forwarding logs

I noted the IP of each VM, then added this rule to `/etc/rsyslog.conf` on the Metasploitable VM:

```
*.* @[SPLUNK_VM_IP]
```

The single `@` sends everything over UDP. In the Splunk web UI (VM IP, port 8000) I created a UDP data input on port 514.

To check it worked, I searched for `host=[METASPLOITABLE_VM_IP]` and the logs were there.

![Splunk UDP input and host search](screenshots/splunk-input.png)

## Problem: the SQL injection didn't show up

I set DVWA's security level to low and ran a basic injection from the SQL Injection module:

```
1' OR '1'='1
```

Searching Splunk for the payload gave me a burst of 433 events, all at the same time. None of them were the attack. They were system logs, mostly from `dhclient`.

The cause was my rsyslog rule. It only forwarded system logs, and the attack was recorded in the Apache access log, which I wasn't sending anywhere. The pipeline looked like it was working because events were arriving, but they were the wrong events.

I added a rule to forward the Apache access logs to Splunk and ran the attack again.

## SQL injection result

Search used:

```
index=main "SQLi"
```

This time it returned only the attack. The log entry showed:

- **Source IP:** 192.168.56.1, my host laptop
- **Response:** HTTP/1.1 200 OK, so the server processed the request
- **Payload:** `1' OR '1'='1` in plain text in the URL parameter, which means DVWA did nothing to sanitize the input

![SQLi search result in Splunk](screenshots/sqli-result.png)

## Brute-force test and alert

I ran a brute-force login attack against DVWA five times to make sure the logging was consistent. This search returned the events from every run:

```
index=main "vulnerabilities/brute"
```

Then I set up a scheduled Splunk alert on that search. It triggers on a high volume of matching events, adds a flag to my dashboard, and sends an email to a set address.

![Alert configuration](screenshots/alert-config.png)

## What I took from it

- Events showing up in a SIEM doesn't mean the right events are showing up. I should have confirmed the Apache logs were being collected before running the attack.
- Using NAT only while downloading and Host-Only afterwards is an easy way to keep a lab isolated.
- The unsanitized payload was readable straight from the access log, so input validation problems are easy to spot once the right logs are collected.
- Writing down what went wrong and how I fixed it made the project easier to repeat.

All attacks were run against my own VM on the Host-Only network.
