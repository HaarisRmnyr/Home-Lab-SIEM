# Home-Lab-SIEM and Log Analysis Project

**Virtual Machine Creation and Configuration**

**Metasploitable 2 VM:**

- I created a new virtual machine in Oracle VirtualBox, selecting Linux as the guest operating system.

- I used the downloaded Metasploitable 2 virtual disk image (.vmdk) to provision the VM, allowing for a pre-configured vulnerable environment.
  Splunk VM:

- I provisioned a second VM, this time using an Ubuntu Server 20.04 ISO file as the installation media. This OS would host the Splunk Enterprise application.
- Initially, the Splunk VM's network adapter was configured with NAT to provide it with internet access, allowing me to download the Splunk Enterprise package        using the wget command.

**Network and Log Integration**
**Network Configuration:**
- After downloading the Splunk software, I reconfigured the Splunk VM's network adapter to Host-Only mode. This created a secure, isolated network between the host   machine (my laptop) and the virtual machines, ensuring the lab could not be accessed from the public internet.

- I obtained the IP addresses of both the Splunk VM and the Metasploitable VM to configure log forwarding.

**Log Forwarding Setup:**

- To successfully link the two VMs, I configured rsyslog on the Metasploitable machine to forward system logs to the Splunk VM.

- I edited the rsyslog.conf file to add a forwarding rule: *.* @[SPLUNK_VM_IP], which directed all log data to the Splunk instance's listener port.

**Verification of Success**
- To verify that the logs were being successfully ingested by Splunk, I performed the following:

- I accessed the Splunk web interface by navigating to the Splunk VM's IP address and the default port (8000) in my browser.

Within Splunk, I created a new UDP data input on port 514 to receive the rsyslog traffic.

A simple search query in Splunk for host=[Metasploitable_VM_IP] confirmed the successful ingestion of logs, demonstrating a functional SIEM pipeline.

**Project 2: SQL Injection Attack Detection**
**Setting up the Test**
To initiate the experiment, I navigated to the Damn Vulnerable Web Application (DVWA) on the Metasploitable VM by entering its IP address into a web browser. Within the DVWA interface, I configured the security level to "low" to ensure the application was susceptible to basic attacks. I then proceeded to the SQL Injection module and conducted a test by submitting a common boolean-based payload: '1' OR '1'='1. The purpose of this test was to trigger a malicious event that would be logged by the web server.

**Initial Findings & Troubleshooting**
Upon performing the attack, I returned to the Splunk search interface and ran a query to search for the malicious payload. I found a sudden burst of 433 events that had occurred simultaneously. However, a detailed analysis revealed that these events were not the SQL injection attempts I was looking for. Instead, they were unrelated system logs from services like dhclient, which were being forwarded from the Metasploitable VM.

This led me to a critical discovery: my initial log forwarding setup was only configured to capture system-level logs, not the application-specific logs from the web server. To correctly capture the attack, a new experiment would have to be conducted after adjusting my Splunk configuration to also ingest the Apache web server's access and error logs.

This troubleshooting phase proved invaluable, as it highlighted the importance of a precise log ingestion pipeline and the need to verify that a SIEM is collecting the correct data.

Following my initial findings that the web application logs were not being forwarded, this phase of the project focuses on correcting the log ingestion pipeline and successfully capturing a malicious SQL injection event. This demonstrates an understanding of log forwarding configuration and targeted threat hunting.

**Configuring Apache Log Forwarding**
- To ensure Splunk could collect the application-level logs where the SQL injection attack was recorded, I configured the Apache web server on the Metasploitable VM to forward its access logs.

- I navigated to the Apache configuration files on the Metasploitable VM.

- I added a new rule to forward all web server access logs to my Splunk instance. This was a critical correction to my previous configuration, which only forwarded system-level logs like those from dhclient.

This step was essential to bridge the gap between my attack and my SIEM, proving the importance of a properly configured log pipeline.

**Re-conducting the Attack & Analyzing Results**

After configuring the new log forwarding, I re-conducted the SQL injection test. This time, a refined search in Splunk successfully isolated the malicious events. The logs provided clear and definitive evidence of the attack, its source, and its impact.

Splunk Query: index=main "SQLi"

This query successfully filtered out all unrelated system chatter, leaving only the precise events related to the attack.

Source of the Attack:

The logs identified the source IP address as 192.168.56.1, which is the IP of my host machine. In a real-world scenario, this would be a key indicator of the attacker's origin.

Impact of the Attack:

The log entry showed an HTTP/1.1 request with a 200 OK status code, confirming that the vulnerable web server successfully processed the malicious payload.

Crucially, the full log entry captured the injected string, 1' OR '1'='1, in the URL parameter. This proved that the application did not sanitize the input, leaving it open to a successful injection.

This project demonstrates my ability to not only detect a threat but also to troubleshoot a complex technical issue and successfully verify the integrity of my security monitoring environment.


I can help you update your README file to document your successful brute-force attack experiment. This will add another strong portfolio piece.

Here is an updated category for your README.md file, which you should add after your previous sections. It is structured to be professional, clear, and to showcase your skills effectively.

**Project 3: Brute-Force Attack Detection & Automated Alerting**

This project demonstrates my ability to detect and automatically alert on brute-force attacks by analyzing a large volume of failed login attempts. It builds on my previous log analysis skills to implement a proactive security measure.

**Simulating the Attack**
I conducted a simulated brute-force attack targeting the DVWA application on the Metasploitable VM. The goal was to submit a high number of incorrect login attempts in a short period to trigger an alert. I performed this test five separate times to ensure consistent and reliable logging.

**Log Analysis and Threat Hunting**
Upon conducting the attacks, I performed a targeted search in Splunk to identify the malicious traffic.

Splunk Query: index=main "vulnerabilities/brute"

Findings: My search successfully returned all events related to the brute-force attempts. The logs confirmed that the requests were directed at the brute-force module of the DVWA application. This verified that my log forwarding and Splunk searches were working correctly.

**Automated Detection and Response**
To demonstrate the implementation of a proactive security control, I created an automated alert to detect and notify me of future brute-force attacks.

- Alert Configuration: I configured a scheduled alert that monitors my Splunk index for a high volume of events containing the string "vulnerabilities/brute."

- Dashboard Integration: The alert was set to add a flag to my dashboard, providing a visual warning of a potential attack.

- Email Notification: I also configured the alert to send an email to a specified address upon detection, simulating a real-world notification to a security analyst for immediate investigation.

This project showcases my proficiency in log analysis and my ability to configure automated, proactive alerts—a key skill for any role in a Security Operations Center (SOC).
