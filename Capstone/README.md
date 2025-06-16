![image](https://github.com/user-attachments/assets/dc69df1b-7cd9-4df8-ad10-9a28431a8354)


# Vulnerability Capstone


## Task 1 : Introduction
  It is your task to perform a security audit on the blog; looking for and abusing any vulnerabilities that you find.
  
Let's get hacking...

<br>

## Task 2 : Exploit the Machine

   I stated the machine and I got the IP i.e. `10.10.170.8`
   
![image](https://github.com/user-attachments/assets/23788df8-f339-4ede-a6ce-7a83a67c7cc8)

<br>

> Initial Reconnaissance
  I began by performing a basic Nmap scan to identify open ports and services on the target machine (`10.10.170.8`)....
      
        nmap 10.10.170.8
        
![image](https://github.com/user-attachments/assets/e4580b70-dbbf-444c-ae6a-d59a0b731871)


<br>

### **What is the name of the application running on the vulnerable machine?**
  
  I try visiting the HTTP service at `http://10.10.170.8`.

![image](https://github.com/user-attachments/assets/5530f38f-a9ef-4d36-9bc2-17ad872a1ddd)

So, the web application name revealed i.e `Fuel CMS`


<br>

### **What is the version number of this application?**

  If we keep looking in the web application... we can find the verson.
![image](https://github.com/user-attachments/assets/eda85b92-82a6-4963-91fe-358878ea5aea)
Our version is `1.4`


<br>

### **What is the number of the CVE that allows an attacker to remotely execute code on this application?**

  Through online research, I identified the relevant vulnerability:  

![image](https://github.com/user-attachments/assets/c4132002-f5b3-4c5d-953e-ed09f90212c0)

`CVE-2018-16763` - A remote code execution vulnerability in Fuel CMS versions 1.4.1 and below.



<br>

### **Use the resources & skills learnt throughout this module to find and use a relevant exploit to exploit this vulnerability.**

  I try to find the exploit in the google specifically enlisted in `exploit-db`.

![image](https://github.com/user-attachments/assets/a91c60cf-2de6-4d2f-ba5d-3f4447dcdc7b)
  
  I copied this RCE code for `CVE-2018-16763` from the [exploit-db](https://www.exploit-db.com/exploits/50477)
  
![image](https://github.com/user-attachments/assets/79f743f0-3e63-4c3d-bd79-24c05cb35696)

  After copying the code, I pasted it into the file I created in my attacking machine.. then I ran the command to check the help manual. 

![image](https://github.com/user-attachments/assets/5c0705a1-25b6-4bf1-a11d-87f378a34901)

  As the manual suggested we can specify the target url by using the `-u` flag. So, I used the command
  
    python3 CVE-2018-16763.py -u http://10.10.170.8/
    
![image](https://github.com/user-attachments/assets/80b419d4-e57e-4d4a-914f-f08f77158322)

However, this only returned `system` responses without proper command execution.


<br>
  After further research, I discovered a more effective exploit from GitHub:

![image](https://github.com/user-attachments/assets/2961243f-8902-4247-bf86-12b270191ee6)

  

![image](https://github.com/user-attachments/assets/8ed00645-e477-4853-9485-81b78146baf4)



<br>


### **What is the value of the flag located on this vulnerable machine? This is located in /home/ubuntu on the vulnerable machine.**

![image](https://github.com/user-attachments/assets/e6ba1026-b648-4ef7-9180-54d3b3fdeb14)



<br>


