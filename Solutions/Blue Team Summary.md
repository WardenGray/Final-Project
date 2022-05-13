## **Table of Contents**

- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic &amp; Behavior

### **Network Topology**

The following machines were identified on the network:

- Hyper V Host Manager
  - **Operating System** : Windows 10
  - **Purpose** : Contains the victim machine and attacker machine
  - **IP Address** :192.168.1.1
- Kali
  - **Operating System** : Kali Linux 2020.1
  - **Purpose** : Used as attacking machine
  - **IP Address** : 192.168.1.90
- Capstone
  - **Operating System** : Linux (Ubuntu 18.04.1 LTS)
  - **Purpose** : Used as a testing system for alerts
  - **IP Address** : 192.168.1.100
- ELK
  - **Operating System** : Linux (Ubuntu 18.04.1 LTS)
  - **Purpose** : Used for gathering information from the victim machine using Metricbeat, Filebeat, and Packetbeat
  - **IP Address** : 192.168.1.100
- Target 1
  - **Operating System** : Linux 3.2 - 4.9
  - **Purpose** : Used as victim machine
  - **IP Address** : 192.168.1.110

### **Description of Targets**

The target of this attack was: Target 1 (192.168.1.110)

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### **Monitoring the Targets**

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

**Excessive HTTP Errors**

Excessive HTTP Errors is implemented as follows:

- **Metric** : Packetbeat: http.response.status\_code \&gt; 400
- **Threshold** : grouped http response status codes above 400 every 5 minutes
- **Vulnerability Mitigated** : Used for intrusion detection/prevention, especially against DDoS and Brute Force attacks. Could be used to block suspicious IPs.
- **Reliability** : Medium reliability. The alert shouldn&#39;t trigger many (if any) false positives. However, an attacker may be able to bypass the alert with a low-and-slow approach, only sending a few status errors per minutes. Such an attack would still show up in the logs but would create a false negative for this alert specifically.

#### **HTTP Request Size Monitor**

HTTP Request Size Monitor is implemented as follows:

- **Metric** : Packetbeat: http.request.bytes
- **Threshold** : The sum of the requested bytes is over 3500 in 1 minute
- **Vulnerability Mitigated** : Used to detect a large amount of http requests which may indicate a DDoS or brute force attack.
- **Reliability** : Medium reliablity. The alert should not any false positives. The threshold is sufficiently high enough such that any alerts will indicate legitimate suspicious activity. As with the prior alert an attacker might be able to avoid triggering this specific alert (thus generating a false negative) by limiting the amount of http requests they send per minute but such an attack could still be detected by other means.

#### **CPU Usage Monitor**

CPU Usage Monitor is implemented as follows:

- **Metric** : Metricbeat: system.process.cpu.total.pct
- **Threshold** : The maximum cpu total percentage is over .5 in 5 minutes
- **Vulnerability Mitigated** : Certain viruses and malware (such as cryptojackers) utilize a high amount of memory to run their own processes. This alert will detect if over 50% of the CPU&#39;s memory is consitently used for a time span lasting 5 minutes or longer.
- **Reliability** : Medium reliability. It is possible this alert will generate false positives if legitimate, memory-hungery processes are running.