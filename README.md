# Home-Lab-SIEM-SetUp
Virtual Machine Creation and Configuration

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
