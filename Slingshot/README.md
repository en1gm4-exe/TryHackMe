![image](https://github.com/user-attachments/assets/06cfd2ea-9cc8-4e42-8f0b-1bdc29c7393b)!

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

First we need to login into the web portal `http://10.10.112.194` with the given credentials..

  ![image](https://github.com/user-attachments/assets/fe733a2f-12e2-4f41-85c5-3b4adc80e6c0)

  So, using these credentails, I was able to login into the portal..

  ![image](https://github.com/user-attachments/assets/d6f24914-9d09-476b-80fa-e2f95f13db17)




1. Attacker's IP

  - Approach:
    - We need to go the `discover` page and we need to add `transactions.remote_address` in the field selection.
    - We have to setup the filter for data `July 26, 2023`
    - Check for unusual IP addresses making multiple requests by clicking on the `field` we added.

      ![image](https://github.com/user-attachments/assets/72a6035f-e50d-4657-b67a-7a6d56bcf357)

    
  
2. First scanner used against the web server
   
  - Approach:
    - After finding the IP, we can simply add that IP in the filter by clicking on the `⊕` sign
    - Since, we looking for the agent for scanner, so we need to add a new field `request.headers.User-Agent` to identify the name.

      ![image](https://github.com/user-attachments/assets/1a42a316-f4c8-4596-9aab-e87c0997d1ec)

    - To look for the earliest scanning activity , we need to sort the cpolumn by `Old-New`

       ![image](https://github.com/user-attachments/assets/88b30914-c2bc-403b-be1d-82c4ac7bdc91)

    - Check for common scanner `user agents` like `Nikto`, `Nessus`, etc.

        ![image](https://github.com/user-attachments/assets/aea4e7e6-0df3-458a-b5d8-e26beb457f0c)




3. User Agent of the directory enumeration tool
    
  - Approach:
    - Mostly in directory scanning we get status code like `503` , `500` , `405` , `404` ,`401` etc, which are different from normal status code we get i.e. `200 OK`. Thus, considering these status code as a factor we can eaily idenify the malicious activity. For status code we need to add a new field `response.status` to identify the status code of the response..
        
      ![image](https://github.com/user-attachments/assets/fbf5b71a-0e81-4a64-aede-67da1676f040)
   
    - Now, we need to sort the results on the basis of the status code, so, I sort it from `High-Low` considering the error or abnormal status code are greater than `200`.
        
      ![image](https://github.com/user-attachments/assets/666765c5-8fdc-48aa-af0f-999f8b4f5fc5)

    - Check the User-Agent header for tools like `dirbuster`, `gobuster`, etc.

      ![image](https://github.com/user-attachments/assets/23309e54-ec9d-434b-8b9f-d86d6c55fd9f)




4. Number of requested resources the attacker failed to find

  - Approach:
    - Now we need to add that status code `404` in the filter by clicking on the `⊕` sign

      ![image](https://github.com/user-attachments/assets/34d77b94-c6a7-4fdf-83ff-ffd222929d2a)

    - Count `404` responses to the attacker's IP
  
      ![image](https://github.com/user-attachments/assets/1e86277b-58c1-41c8-96da-c4a9eed082ca)
     

5. Flag under the interesting directory
  
  - Approach:
    - Now we need to add status code `200` and agent `Gobuster` in the filter by clicking on the `⊕` sign

      ![image](https://github.com/user-attachments/assets/cb2be885-fce5-4931-84f4-2f914e679c15)
         
    - Check for flag-like strings in response or directory

      ![image](https://github.com/user-attachments/assets/f1d0f8ab-6537-4fb5-8a05-c8c4f02392ed)

        

6. Login page discovered

  - Approach:
    - I tried looking into the directories with `200` status code but there wasn't any login directory. So, I tried to edit the filter hoping if there is any redirection status code `301`  

      ![image](https://github.com/user-attachments/assets/da0fd71c-e2f3-4cb7-a9b1-c710a3aca6b4)


    - Look for admin/login related paths
    - Check successful `200` responses after enumeration

      ![image](https://github.com/user-attachments/assets/e101f601-08ac-45de-a16c-d3095f5e8ccf)

    So, we can deduce the login page was..

      ![image](https://github.com/user-attachments/assets/ad315c42-ece9-41ec-bbcc-0d11bb313a56)


7. User agent of the brute-force tool
      
  - Approach:
    - I simply reset the user agent filter from `Gobuster` 
    
      ![image](https://github.com/user-attachments/assets/35fc1204-8ece-48c0-b255-ead6f27ce918)


    - Check User-Agent for tools like `hydra`, `wfuzz`, etc.

      ![image](https://github.com/user-attachments/assets/a76d90ca-ec99-4332-b7fb-0eb779074cc0)
    

8. Username:password combination used

  - Approach:
    - I add the status code filter to `200` and agent filter `Hydra` by clicking on the `⊕` sign

      ![image](https://github.com/user-attachments/assets/c8d1c996-79be-4345-92ca-72d46fe96d61)

    - Look for successful login (`200` after `POST`)
  
      ![image](https://github.com/user-attachments/assets/1017264a-300c-45a3-b967-353bc655bf47)

  
    - Check credentials in POST data, for doing so, we need to click on this button to get the details..

      ![image](https://github.com/user-attachments/assets/a366af2d-82ec-48ea-a40f-791092edaba7)

    - So, looking into the request detail we found ourself a encoded string.

      ![image](https://github.com/user-attachments/assets/5a79692f-9213-47bd-81d6-4f1e5d91dc29)

    - To get the password, I simply used the [CyberChef](https://gchq.github.io/CyberChef/)..

      ![image](https://github.com/user-attachments/assets/6a17c6d4-880c-42b0-b0d7-23ef1b01ced4)

    

9. Flag in uploaded file
    
  - Approach:
    - I add a new field `http.method` and set-up the filter for `POST` requests.

      ![image](https://github.com/user-attachments/assets/53ff992d-899f-41b9-b559-317dded8ec43)

      
    - By looking for file upload activity, there were total of 4 POST and I figuered out the malicious one..
      
      ![image](https://github.com/user-attachments/assets/df5ccd39-a6ec-4dd0-9443-9d4df2f7a04f)

                  
    - To check contents of uploaded files we need to check the details..
      
      ![image](https://github.com/user-attachments/assets/9c2fbc52-1cb5-4210-848f-1c6872bd45f2)


10. First command ran on the web shell
      
  - Approach:
    - By looking into the details in the last task we identified the file name that was uploade by the attacker. So, I simply searched it . Along, I set-up the filter for `200` status code response.

      ![image](https://github.com/user-attachments/assets/18da66f6-c602-49c0-bc41-b5c27b6086bd)


    - So, look into the result we can find the command execution parameter..

      ![image](https://github.com/user-attachments/assets/25568a18-010d-4d6e-a474-38f9e5865d55)

    



11. File location with database credentials
      
  - Approach:
    - Looking for LFI attempts, I was usign the filter `http.url :` for common directories where attacker can attempt LFI, So I found some attempt of the LFI in admin directory path.. 

      ![image](https://github.com/user-attachments/assets/7d6ee6a9-7bb9-4cb7-8691-2ddbefdaa8b1)

    - Common config files with credentials

      ![image](https://github.com/user-attachments/assets/7e7ba665-0f82-463f-9084-117d60064f95)


12. Directory used to access database manager
      
  - Approach:
    - Look for `phpMyAdmin` or any simillar path, but we can see our directory is `phpMyAdmin`
   
      ![image](https://github.com/user-attachments/assets/f925cf16-63e9-4dd0-a525-c32b4ede9924)

    
        
13. Name of the exported database
      
  - Approach:
    - Looking for export/backup activity, I simply filter out the `export` directory/file with wildcard  as `export*`

      ![image](https://github.com/user-attachments/assets/afe5eef8-549f-44ac-b424-c6013c879319)

      
    - After this, We got ourself a `export.php`

      ![image](https://github.com/user-attachments/assets/c090a2e5-41f1-4838-b44c-fcc01481ca3d)


    - So, I simply checked the details of it..

      ![image](https://github.com/user-attachments/assets/a0d10046-9456-44ca-94b4-9458d1e16c87)

    - Just looking into the request body we found out `DB`(database) parameter.



14. Flag inserted into the database

  - Approach:
    - Again, I started filtering with wildcards but by simple `http.url:/phpmyadmin/*`, I was unable to get any file related to database. So, I attempted to search with the wildcard for `.php` extension  as `http.url:/phpmyadmin/*.php`
      
      ![image](https://github.com/user-attachments/assets/2996fdab-fdbb-4807-8e51-2acdb198f135)

    - I got multiple matches for the above query..

      ![image](https://github.com/user-attachments/assets/b2e742af-2752-42c1-9c16-c966cb3b3aca)
 
    
    - So, Make it more clear, I filter on the basis of content. As we know attacker inserted something into database so, Using `SQL INSERT` as filter I found some suspicious results..

      ![image](https://github.com/user-attachments/assets/d86dc016-9f2b-4760-8b36-122f4cabd3cd)


    - I start looking into the message parameter in the details of the `import.php`, where I found the entry...

      ![image](https://github.com/user-attachments/assets/54433ec6-dec5-4896-816a-3352493eed6d)

    - Looking into the `INSERT` request we can identify the `VALUES` SQL parameter, which shows the inserted content must be just next ahead of it.



<br>



What was the attacker's IP?
      
      10.0.2.15

What was the first scanner that the attacker ran against the web server?

      Nmap Scripting Engine

What was the User Agent of the directory enumeration tool that the attacker used on the web server?

      Mozilla/5.0 (Gobuster)

In total, how many requested resources on the web server did the attacker fail to find?

      1867

What is the flag under the interesting directory the attacker found?

    a76637b62ea99acda12f5859313f539a

What login page did the attacker discover using the directory enumeration tool?

    /admin-login.php

What was the user agent of the brute-force tool that the attacker used on the admin panel?

    Mozilla/4.0 (Hydra)

What username:password combination did the attacker use to gain access to the admin page?

    admin:thx1138

What flag was included in the file that the attacker uploaded from the admin directory?

    THM{ecb012e53a58818cbd17a924769ec447}

What was the first command the attacker ran on the web shell?

    whoami

What file location on the web server did the attacker extract database credentials from using Local File Inclusion?

    /etc/phpmyadmin/config-db.php

What directory did the attacker use to access the database manager?

    /phpmyadmin

What was the name of the database that the attacker exported?

    customer_credit_cards

What flag does the attacker insert into the database?

    c6aa3215a7d519eeb40a660f3b76e64c


<br>
