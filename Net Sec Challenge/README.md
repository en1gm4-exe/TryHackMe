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
- Used Nmap to perform a SYN scan (-sS) on ports 1 through 10,000 (-p1-10000).
- The -T4 flag was added for faster scanning.

**Command:**
      
      nmap -sS -T4 -p1-10000 10.10.68.75
**Output:**
![image](https://github.com/user-attachments/assets/571984da-e84c-482a-a4e8-efef72cdc5cb)
**Answer:** The highest open port below 10,000 is `8080`.






![image](https://github.com/user-attachments/assets/01cafa28-93d8-4718-879f-0cefb75fa55a)

![image](https://github.com/user-attachments/assets/6d01b7b2-f060-43e2-8d97-3a6b802d04ca)

![image](https://github.com/user-attachments/assets/e0f73f18-13d0-4e24-8ba1-0ac18b19bd60)

![image](https://github.com/user-attachments/assets/9f385f27-37eb-4a2d-8ff2-a2d8564d537d)

![image](https://github.com/user-attachments/assets/d7f884dd-e01d-4687-8899-3f4c5c442635)


q-7
![image](https://github.com/user-attachments/assets/c047c703-d860-4b91-b5dd-d68087932bc3)
![image](https://github.com/user-attachments/assets/3d92fd3a-ba38-4ece-b6f3-8a1b8b301178)
![image](https://github.com/user-attachments/assets/13e7ade7-b2ae-4f56-bec6-62e3d5184c1b)



q-8
![image](https://github.com/user-attachments/assets/6e70c5f1-7d65-4773-8423-b650bc9a51cb)
![image](https://github.com/user-attachments/assets/aa5c9b15-6e3b-43b3-8fac-ec609dd19e1e)



