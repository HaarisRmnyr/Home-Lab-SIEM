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
