![image](https://github.com/user-attachments/assets/27914ca8-563f-476e-8122-ebbe842200e5)


# Net Sec Challenge

## Introduction
   This walkthrough details my approach to solving the Network Security Challenge room on TryHackMe. The challenge tested various network security skills using tools like Nmap, Telnet, and Hydra. Below, I'll break down each task with detailed explanations and commands used.

<br>

## Task 1: Setup
   Before starting, I launched the AttackBox and the target VM with the following details:
Target IP Address: 10.10.68.75
![image](https://github.com/user-attachments/assets/f8fad8a2-076b-41a6-ac36-89c8c501f4ea)



## Task 2: Challenge Questions

### Q-1: Highest Port Number Open Below 10,000

**Objective:** Identify the highest port number below 10,000 that is open on the target machine.

**Approach:** 
- Used Nmap to perform a SYN scan (`-sS`) on ports 1 through 10,000 (`-p1-10000`).
- The -T4 flag was added for faster scanning.

**Command:**
      
      nmap -sS -T4 -p1-10000 10.10.68.75
**Output:**
![image](https://github.com/user-attachments/assets/571984da-e84c-482a-a4e8-efef72cdc5cb)
**Answer:** The highest open port below 10,000 is `8080`.

<br>

### Q-2: Open Port Above 10,000
**Objective:** Find an open port outside the common range (above 10,000).

**Approach:**
- Scanned ports above 10,000 using Nmap (`-p10000-`).

**Command:**
            
         nmap -T4 -p10000- 10.10.68.75
            
**Output:**

![image](https://github.com/user-attachments/assets/01cafa28-93d8-4718-879f-0cefb75fa55a)

**Answer:** The open port above 10,000 is `10021`.

<br>

### Q-3: Total Number of Open TCP Ports
**Objective:** Determine how many TCP ports are open on the target.

**Approach:**
- Performed a TCP connect scan (`-sT`) on ports 1 through 10250 (`-p1-10250`).

**Command:**

      nmap -sT -T4 -p1-10250 10.10.68.75

**Output:**
![image](https://github.com/user-attachments/assets/6d01b7b2-f060-43e2-8d97-3a6b802d04ca)
**Answer:** There are `6` open TCP ports.

<br>

### Q-4: Flag Hidden in HTTP Server Header
**Objective:** Retrieve the flag hidden in the HTTP server header.

**Approach:**
- Used curl with the `-I` flag to fetch the HTTP headers.

**Command:**

      curl http://10.10.68.75 -I
      
**Output:**
![image](https://github.com/user-attachments/assets/e0f73f18-13d0-4e24-8ba1-0ac18b19bd60)

**Answer:** The flag is `THM{web_server_25352}`.

<br>

### Q-5: Flag Hidden in SSH Server Header
**Objective:** Find the flag hidden in the SSH server header.

**Approach:**
- Used telnet to connect to the SSH service on port `22`.

**Command:**

      telnet 10.10.68.75 22

**Output:**
![image](https://github.com/user-attachments/assets/9f385f27-37eb-4a2d-8ff2-a2d8564d537d)
**Answer:** The flag is `THM{946219583339}`.


<br>

### Q-6: FTP Server Version
**Objective:** Identify the version of the FTP server running on a non-standard port.

**Approach:**
- Ran a service version detection scan (`-sV`) on port 10021.

**Command:**

      nmap -T4 -sV -p10021 10.10.68.75

**Output:**
![image](https://github.com/user-attachments/assets/d7f884dd-e01d-4687-8899-3f4c5c442635)
**Answer:** The FTP server version is `vsftpd 3.0.5`.


<br>

### Q-7: Flag Hidden in FTP User Account
**Objective:** Retrieve the flag hidden in one of the FTP user accounts (`eddie` or `quinn`).

**Approach:**
- Used [Hydra](https://www.hydradongle.com/download/software) to brute-force passwords for the usernames eddie and quinn.
- Logged into the FTP server to locate the flag.

**Hydra Commands:**

      hydra -l eddie -P /usr/share/wordlists/rockyou.txt ftp://10.10.68.75:10021 -t 4
  
      hydra -l quinn -P /usr/share/wordlists/rockyou.txt ftp://10.10.68.75:10021 -t 4

**Output:**

![image](https://github.com/user-attachments/assets/c047c703-d860-4b91-b5dd-d68087932bc3)

<br>

**Command:**

      ftp 10.10.68.75 -p 10021

> **Eddie-FTP**

   Entering `eddie` as the username and `jordan` as the password, I tried looking into files but eddie don't have the flag.
   
![image](https://github.com/user-attachments/assets/3d92fd3a-ba38-4ece-b6f3-8a1b8b301178)

> **Quinn-FTP**

   Entering `quinn` as the username and `andrea` as the password, I tried looking into files and there was the file named ftp_flag.txt. So, I downloaded it to my local machine using command
      
      get ftp_flag.txt   

   And after reading the content in the file, I got the my flag.
      
      cat ftp_flag.txt

![image](https://github.com/user-attachments/assets/13e7ade7-b2ae-4f56-bec6-62e3d5184c1b)


**Answer:** The flag is `THM{321452067098}`.


<br>


### Q-8: Flag from HTTP Challenge on Port 8080
**Objective:** Solve the challenge hosted at `http://10.10.68.75:8080` to retrieve the flag.

**Approach:**

1. Initially, a regular Nmap scan triggered the IDS.

**Command:**

      nmap -sN 10.10.68.75

**Output:**
The challenge page displayed:
![image](https://github.com/user-attachments/assets/6e70c5f1-7d65-4773-8423-b650bc9a51cb)
   So, I can see it was detected and IDS detection is working fine with my `Kali linux`.

2. Used a Null scan (-sN) to avoid detection.

**Command:**

      nmap -sN 10.10.68.75

**Output:**
The challenge page displayed the flag.:
![image](https://github.com/user-attachments/assets/aa5c9b15-6e3b-43b3-8fac-ec609dd19e1e)

**Answer:** The flag is `THM{744399}`.


