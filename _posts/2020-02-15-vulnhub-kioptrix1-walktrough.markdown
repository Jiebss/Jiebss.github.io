---
title:  "Vulnhub: Kioptrix 1 — Walkthrough"
date:   2020-02-15 15:30:33
---

![](/assets/4/1.png)

I’ve set a decent amount of goals for myself to achieve, one of those is OSCP eventually. Overtime i’ve met some amazing people who actually recommended me doing the Kioptrix serie of vulnerable machines since they’re closely related to what the OSCP exam will be like. With that being said, i decided to give it a shot!

<h2>Description</h2>
This Kioptrix VM Image are easy challenges. The object of the game is to acquire root access via any means possible (except actually hacking the VM server or player). The purpose of these games are to learn the basic tools and techniques in vulnerability assessment and exploitation. There are more ways then one to successfully complete the challenges.

<h2>Enumeration</h2>
I ran netdiscover the find the IP associated with the vulnerable machine. Once i found the IP, i added this to my /etc/hosts file with an alias.

Once that was done, let’s start off with our nmap scan.

```
nmap -sC -sV -oA scans/nmapinitial kioptrix1.vh
```

![](/assets/4/2.png)

When going through our nmap scan we can see that there are a bunch of interesting ports. I noticed the apache version, which is seriously outdated and mod_ssl running on port 443. I decided to take a look into that and found the OpenF**k exploit, which can be found by click [here](https://github.com/heltonWernik/OpenLuck)

Great, let’s move onto exploitation!

<h2>Exploitation</h2>
Let’s go ahead and download that exploit to our machine.

```
git clone https://github.com/heltonWernik/OpenFuck.git
```

![](/assets/4/3.png)

We have to make sure that libssl-dev is installed.

```
apt-get install libssl-dev
```

![](/assets/4/4.png)

Once done, move to the OpenF**k directory and compile it.


```
gcc -o OpenFuck OpenFuck.c -lcrypto

```
![](/assets/4/5.png)

And finally we are able to run the exploit!

![](/assets/4/6.png)

Since the VM is running RedHat with apache version 1.3.20, we are going to select 0x6a or 0x6b. You can see the full list of targets when running the exploit, the screenshot shows only a small amount.
Now that we have all the information we need, let’s exploit this machine!

![](/assets/4/7.png)

Great, we have a shell running with root privileges. Last thing to doo is get that flag!

Let’s quickly spawn a TTY shell.

```
/bin/sh -i
```
![](/assets/4/8.png)

After looking aroud for a while i found the flag in /var/mail.

![](/assets/4/9.png)

<h2>Final words</h2>
And there we have it..! The first box of the kioptrix serie completed. In all honesty, the VM was rather easy. Not a lot of enumeration besides our nmap scan. My eye almost immediatly saw the highly outdated Apache version so there was no reason to do any further enumeration. Atleast not untill after i tried that possibility. Luckily there are 4 more levels and they will only get harder.

Thanks for reading and happy hacking!