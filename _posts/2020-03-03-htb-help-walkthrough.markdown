---
title:  "Hack The Box: Help — Walkthrough"
date:   2020-03-03 12:17:54
---

Hack The Box machine 'Help' was retired some time ago meaning that i'm allowed to post my write up about it. I've actually rooted this box last year, but never had the time to properly do a write up about it.

![](/assets/5/1.png)

<h2>Enumeration</h2>
Let’s start off with our nmap scan.

```
nmap -sC -sV -oA scans/nmap_initial 10.10.10.121
```

![](/assets/5/2.png)

Allright, there is something running on port 80. Let's check that out. 
We are looking at the default apache page, let's bruteforce for directories in case we are missing something.

```
dirb http://10.10.10.121
```

While dirb is running we're going to take a look at the service running on port 3000.

![](/assets/5/3.png)

I played around with it for a bit but didn't give us too much information.
Luckily dirb has finished and it gave us some results, let's check it out.

![](/assets/5/4.png)

We can see 2 other directories being listed, checked them both out. /support looks promising!
Enough recon, let's start exploiting this!

<h2>Exploitation</h2>
I went to exploit-db and searched for HelpDeskZ exploits, 2 were listed.

I used the arbitary file upload exploit which you can find by clicking [here](https://www.exploit-db.com/exploits/40300).

Go to 'Submit a ticket', check general and press Ok. We can submit a ticket here and attach a file exactly as the exploit describes, great!

Here is were it gets a bit tricky. I've had a bunch of struggles getting this exploit to work.
I went to the official github page which you can find by clicking [here](https://github.com/evolutionscript/HelpDeskZ-1.0/blob/master/controllers/submit_ticket_controller.php) and looked at the code for the file upload because when i upload a php file it says 'file not allowed'.

While going through the code i found 2 interesting things:
a) the timezone needs to match that from the remote server.
b) the file is uploaded in baseurl/uploads/tickets

To check the timezone of the remote host simply do
```
curl -v 10.10.10.121
change your timezone on your local machine to the same timezone
```

Once done, retry uploading your reverse php shell. It'll give you the same error (file not allowed), but it did get uploaded!

Save the exploit code on your attacker machine and run it.

```
python exploit.py http://10.10.10.121/support/uploads/tickets/ shell.php
```

![](/assets/5/5.png)

After waiting for a short amount of time it'll give you the shell url.

Listen on your local machine to the appropriate port.

```
nc -lvp 1234
```

Run the reverse shell either by visiting the link or with curl.

![](/assets/5/6.png)

Great, we have our reverse shell! Lets spawn a tty shell. We also found the user flag after looking around for a bit.

![](/assets/5/7.png)

Let's continue to root this box!

<h2>Privilege Escalation</h2>
Took me a while but priv esc on this box is rather common, guess it was a late one.
I looked up the kernel.

```
uname -mrs
```

Interesting! We found a known exploit for this kernel version, which you can find by clicking [here](https://www.exploit-db.com/exploits/44298).
Let's start a werbserver to get this on our remote server.

```
python -m SimpleHTTPServer 4000
```

After that, we can use wget it to get it on our remote machine.

![](/assets/5/8.png)

Once done, we can run the exploit using:

```
gcc evil.c
```

When it's done running, type ls and you'll see an output file named 'a.out', lets check it out.

![](/assets/5/9.png)

Great, we have root privs! Lets search for the flag!

![](/assets/5/10.png)

Thank you for reading and happy hacking!