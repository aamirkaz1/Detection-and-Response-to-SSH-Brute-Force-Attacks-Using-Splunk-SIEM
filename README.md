## Objective
The goal of this project was to design and implement a local cybersecurity home lab to simulate a real-world cyber attack and develop detection capabilities. Specifically, I set up a network environment to execute an SSH brute-force attack against a Linux target and utilized Splunk Enterprise to ingest the logs, write detection queries, and build an alerting dashboard.

## Theory 
SSH : Secure Shell is a Cryptographic protocol which allows users to login and control remote servers via command line

Brute Force Attack : Every possible combination of letters, symbols and numbers (usually automated to speed up the process) is known as a brute force attack
Eg: John the ripper, Aircrack ng

Splunk : Splunk is a platform that ingest, indexes and analyzes machine generated data from a source in the real time. Splunk is a provider of SIEM technology Siem is security information and event management.

## Network Architecture
<img width="1200" height="794" alt="image" src="https://github.com/user-attachments/assets/2ebf16a4-02f4-40ab-9583-65e361d3cc64" />


## The lab environment consists of three primary virtual machines hosted on Oracle VirtualBox, connected via an isolated NAT network:
* **Attacker Machine:** Kali Linux (Used to run the Hydra brute-force tool)
* **Target Machine (DMZ):** Ubuntu Server (Configured with SSH exposed to the internal network).Demilitarised zone it is a sub network which acts as a buffe, which exposes external services like web, email, etc. Keeping internal network isolated and safe. Windows and Ubuntu servers with Splunk universal forwarder installed helps in sending alerts to Splunk servers.
* **SIEM Server:** Ubuntu Server running Splunk Enterprise (Configured to receive and index logs from the target).Splunk is installed on Ubuntu and fetches the Suricata logs. Suricata is a subset under SIEM. SIEM is a vast and takes data from various sources which would include firewalls, applications, Suricata ,etc.
* **Firewall:** pfsense (packet filtering). We can also use Palo Alto but pfsense is compatible with Suricata. Integration is easy with its rules. Firewall sends locks to splunk server.
* **Applications:** This block represents the modern, cloud-native services that a company would host and that an attacker would try to hack. However for my current project (detecting SSH brute force), I do not need Docker, Kubernetes, or NGINX at all. 



## Tools & Technologies Used
* **Hypervisor:** Oracle VirtualBox
* **SIEM:** Splunk Enterprise
* **Operating Systems:** Kali Linux, Ubuntu Server
* **Attack Tools:** Hydra
* **Log Forwarding:** Splunk Universal Forwarder

## Methodology & Steps

### 1. The Attack (Red Team Phase)
To simulate the attack, I targeted the SSH service running on the DMZ Ubuntu machine. I used `Hydra` on Kali Linux alongside the `rockyou.txt` password list to brute-force the credentials. 
> **Command used:** `hydra -l [username] -P /usr/share/wordlists/passlist.txt ssh://[target_IP]`

<img width="614" height="303" alt="image" src="https://github.com/user-attachments/assets/50d3d0d1-a225-46d4-8e10-45ba98c3b2aa" />


### 2. Log Ingestion & Forwarding
I installed the Splunk Universal Forwarder on the target Ubuntu machine. I configured the `inputs.conf` file to monitor the `/var/log/auth.log` file (which records SSH login attempts) and forward those logs to the main Splunk SIEM server.

### 3. The Detection (Blue Team Phase)
Once the logs were flowing into Splunk, I wrote a Splunk Processing Language (SPL) query to identify the brute-force behavior. The query looks for a high volume of failed login events (`sshd` failed passwords) originating from a single IP address within a short time frame.

> **SPL Query:** > `index=main sourcetype=syslog process=sshd "Failed password" | stats count by src_ip | where count > 5`

<img width="566" height="97" alt="image" src="https://github.com/user-attachments/assets/ac5f0f4f-a871-4efc-b81c-0494596cf9e1" />

### 5. Alerting & Dashboards
To make the detection actionable, I converted the SPL query into a real-time alert and built a dashboard to visualize:
* Top targeted usernames
* Spike in failed login attempts over time
* The IP address of the attacker
  
Multiple Failed password events for the same user from one src_ip

### 4. Response
Block attacker IP:

<img width="613" height="102" alt="image" src="https://github.com/user-attachments/assets/5afc7cc5-da2a-4c6e-adc0-7285d8960f59" />


## Conclusion & What I Learned
This project provided hands-on experience with the full lifecycle of a basic cyber attack and defense scenario. I gained practical skills in:
* Configuring virtual networks and isolating lab environments.
* Executing automated dictionary attacks using industry-standard tools.
* Forwarding system logs securely to a centralized SIEM.
* Writing custom SPL queries to detect anomalous network behavior.
