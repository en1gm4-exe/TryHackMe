![image](https://github.com/user-attachments/assets/fa913a28-68fd-4973-a49f-e59312b86013)

# Slingshot
  
  Can you retrace an attacker's steps after they enumerate and compromise a web server?


TryHackME Room [link](https://tryhackme.com/room/slingshot)

<br>


## Task 1 : Scenario

 
> Attack Scenario

The attacker targeted `Slingway Inc.`'s e-commerce web server, performing reconnaissance, enumeration, brute-forcing, and ultimately compromising the system. The attack timeline begins on `July 26, 2023`, and involves:

  1. Scanning and enumeration of web directories
  2. Discovery of an admin login page
  3. Brute-force attack on the admin panel
  4. Uploading a web shell
  5. Exploiting Local File Inclusion to get database credentials
  6. Accessing and exporting database contents
  7. Inserting a flag into the database

> Workflow..

