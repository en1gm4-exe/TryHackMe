  ![Banner](banner.png)
# Security Footage 
  <i>Perform digital forensics on a network capture to recover footage from a camera.</i> <br>
  
  [Link](https://tryhackme.com/room/securityfootage)

  

## Task 
<a>Someone broke into our office last night, but they destroyed the hard drives with the security footage. Can you recover the footage? </a>
<br>
<br>

<a>We are given a `PCAP` file for forensics. i.e. [PCAP](https://github.com/en1gm4-exe/TryHackMe/blob/main/Security%20Footage/security-footage.pcap)<br>

<a>First I opened the `.pcap` file in the [wireshark](https://www.wireshark.org/download.html) And I started inspecting the packets. First packet was TCP hand shake intializer, `TCP SYN` packet... </a>  <br>
![image](https://github.com/user-attachments/assets/fa7a3885-517a-440d-bbca-91ac8ad8e843)


<a> Second packet is `TCP ACK` in the hand shake... </a>
![image](https://github.com/user-attachments/assets/34577f35-6f61-41a6-8d05-63e6ccd65deb)

<a> Third packet shows the successful `TCP` hand shake... </a>
![image](https://github.com/user-attachments/assets/39b86186-b333-4f67-bb82-219ca3224ef6)


<a> Forth packet shows the `HTTP GET` request from the attacker... </a>
![image](https://github.com/user-attachments/assets/21fcd182-5884-4730-8cc0-1cdef8431552)

<a>After packet there is no more http protocol packet, so this means rest of the packets are the data tranfer to the attacker after that http request. </a>
![image](https://github.com/user-attachments/assets/57ddcf28-d3a9-496d-93f8-10e79d111b05)

<a>So, I followed the TCP Stream to have a better look at sequential communication or packet transfer.  To open the stream, <br>
- Right Click on any packet
- Then, hover the pointer on `Follow`
- Click on the `TCP Stream` <br>

Or  alternatively you can press `ctrl+alt+shift+T`  </a>
![image](https://github.com/user-attachments/assets/a5d0842b-7b8a-4179-b65c-e59b1454ab37)

<br>
<br>

<a>Once we are in the tcp stream tab, we can notice that there is only one stream in all and on the left side, there is `1 turn` between client and server, so whole communication started after that one `HTTP GET` message.</a>
![image](https://github.com/user-attachments/assets/0dda1d98-0021-4336-b869-98080c7aa0d7)


<a>So, let's take a look on the stream..</a>
![image](https://github.com/user-attachments/assets/acae06f9-3aac-4f21-9710-5c26987cdc9c)

<a> By setting the filter `show as` of the wireshark to `ASCII` , we can see the packet and we can identify the things happening in it.
So, at top we have `HTTP GET` and then we got `200 OK` as server response to the request and the dat tranfer is done in the images as `Content-type` parameter shows the file type is images with extension of `JPEG`.
</a>


<br>
<br>


<a> So, I used my python based tool to extract the data from the `.pcap`. <br>
Tool's walkthrough is present on the tool's repo i.e. [PCAP_Extractor](https://github.com/en1gm4-exe?tab=repositories), for instance using cli version of my tool, I extracted the `JPEG` files using the command <br>
`python3 PCAP_Extractor.py security-footage-1648933966395.pcap -o Flag --jpg`
<i>`-o` is used to name the output directory and `--jpg` is used to tell the tool to scan for jpg files only, since jpg and jpeg are interchangeable extensions. </i>
![image](https://github.com/user-attachments/assets/0a52d48a-7b76-498d-b0c3-6b2c7541e9d8)
![image](https://github.com/user-attachments/assets/8dba49cf-f981-43a6-bfa7-3a33f2b31419)




<a>We succesfully extracted the data from the `.pcap` and looking into the output director, I found the flag is separated in pixels so seeing the image in flow will help to get the flag. </a>
![image](https://github.com/user-attachments/assets/7c2b2831-96ce-4039-bf12-e4dd3c6165b4)

<a>By looking all the `541` images one by one, we can retrieve the flag. </a>



## Flag 🚩
<i> This room is not much complex, so you must try to do it by your own way to retrieve the flag..</i>


