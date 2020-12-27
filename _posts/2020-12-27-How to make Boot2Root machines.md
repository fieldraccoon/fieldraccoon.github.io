---
title: How to make Boot2Root machines for hackthebox / tryhackme
author: fieldraccoon
date: 2020-12-27
categories: [examples]
tags: [hack-the-box, tryhackme, tutorial]
---
# Setup
The idea of me making this machine was to learn how it works, the setup process. Making something vulnerable and eventually how to submit and export my image to the platforms.
## This box consists of:
- ` Nmap` the box to find that port `21` is open
- connecting via `FTP` 
- using `get` to grab a file that contains credentials
- Using those credentials to login via `ssh`
- using `sudo` rules to priv esc to `root`
### This taught me many things involving:
- how to setup an `ssh` server
- How to setup a `ftp` server using `vsftpd`
- Getting used to `systemctl` and `service` to manage the services
- And overall learning how to make one of these machines

So to start with i went and grabbed a base ubuntu image that we will make our machine with. Some people like to use other Linux distributions and windows to make vms but I've stuck with ubuntu to develop my box inside.
You can get an image from [https://ubuntu.com/download](https://ubuntu.com/download). I went with Ubuntu 16.04.

Now to bear in mind at the end we will export this virtual machine into either an `.OVF` or the `.vmdk` for VMware. There are a few that work but your Hypervisor needs to be able to export to an image. I used VMware workstation 16 Pro but i believe that VirtualBox works too so either way make sure you have some sort of software that can do this(there might be some online ones as well but I'm not 100% sure).

### So now we begin our setup for the machine
- First download your image
- Then import it into your hypervisor
- then when prompted make a user(this part doesn't matter you can always remove the user you created later if you don't want them on the machine)

ok now we are good to go.
## planning the machine
This part is always important. I had the idea that i wanted it to be very simple in order to see what works.
I decided on having an ftp server open which gave you some sort of credentials. I then wanted to use those to ssh into the user. I then wanted to have a simple priv esc so i decided on the amazing `sudo -l` go brr.

# Making the machine(user)
## Ftp
So first i needed to setup an ftp server.
After a few quick googles i found that `vsftpd` is the thing we need to use.
So i looked into using that.
I installed it using :
```bash
sudo apt-get install vsftpd
```
I came across this website which was very useful throughout our whole journey. [website-digitaocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-for-a-user-s-directory-on-ubuntu-18-04).
The website says we also need to enable ftp using the firewall.
```bash
sudo ufw allow 20/tcp
sudo ufw allow 21/tcp
```
The port 20 is for data-transfer. 
Anyway at this point i like to get a simple bash script started to maintain my progress but also to make sure if i screw something up i can run the script and everything is back where it should be.
We add a simple `sudo apt-get update` to the start of our file to make sure the system always stays up to date and all the packages work correctly
So our file so far looks like so:
```bash
sudo apt-get update

sudo apt-get install -y vsftpd

sudo ufw allow 20/tcp
sudo ufw allow 21/tcp
```
The `-y` parameter is so that it doesn't prompt us for a download.
Ok all looks good so far.
We can enable FTP to start running with a few commands.
```bash
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
```
And there we go we can login to ftp, however this is only with our normal user and we want to allow anonymous access. To change this we need to edit the default configuration file which is stored in `/etc/vsftpd.conf` 
I add the following lines to the conf file.
```bash
anonymous_enable=YES
local_enable=YES
```
This was to enable anonymous access to our share.
But if you try connecting from here you will realise that when you try running `ls` it is empty. running `pwd` reveals `/` so we know that there is absolutely nothing on here.

So i know added more features to my `conf` file to make this work.
I first made a user on my system called `ftpuser` which you can create with `sudo useradd $username` 


Then created a file to add our ftp data to with `sudo  mkdir -p /var/ftp/pub` and gave it the appropriate permissions(`sudo  chown nobody:nogroup /var/ftp/pub`).

We then simply added whatever we wanted(in our case the creds) to a file in this directory and it will be served on the ftp.
One more thing however we needed to edit the config to make this the root directory for when the anonymous user connects.
```bash
anon_root=/var/ftp/pub/

hide_ids=YES
```
We set the default directory to `/var/ftp/pub` and we also chose to enable the `hide_ids` which displays the user as `ftp` rather than `john` so it doesn't spoil the box.

Ok and this seems to be all for the FTP section of the box. I ran into a few times where it didnt work but it can be restarted successfully with `sudo service vsftpd restart`
Ok now we just need to add a file we want to host on FTP. I included some credentials.

```bash
echo "john:P@ssw0rd" | sudo tee /var/ftp/pub/creds.txt
```
And now we are all ready to go! If you restart ftp and test this locally it works!

# SSH

Now we need to enable Ssh. This part was quite simple as ssh simply works with any user thats already on the machine.
We still include the relevant commands inside our script if for some reason something isnt working.
```bash
sudo apt-get install -y openssh-server 
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.factory-defaults
sudo chmod a-w /etc/ssh/sshd_config.factory-defaults
sudo systemctl restart ssh
```
These commands install ssh, make a backup copy of our config, and gives it permissions so that it cant be tampered with. It also restarts the service so everything is nice and fresh when we want to use it.
From here we simply ssh in with `john`. (`john` was made just like `ftpuser` and of course added with the password we found in `creds.txt`)

We now have user access yay.
# Root
For root I decided to take a simple `sud`o rule exploitation to test things out.
I could have added the user john and had him be able to run everything as root but that would be boring.
Instead i decided to do something even more boring but it took me 1 minute so its fine.

I add john with the permissions to be able to run `cat` as root therefore meaning he can read the root flag as sudo.
```bash
john     ALL=(ALL) NOPASSWD:/bin/cat
```
This is my `/etc/sudoers` file which is where we can modify the sudoers rule.

We know add some appropriate permissions to our files and add the root and user flags with their permissions aswell with the following code:
```bash
sudo bash -c 'echo "flag{i_hope_this_worked}" > /home/john/user.txt'
sudo chown john:john /home/john/user.txt
sudo chmod u+rx /home/john/user.txt
sudo chmod u-w /home/john/user.txt
sudo chmod go-rwx /home/john

sudo bash -c 'echo "flag{root_flag_poggers}" > /root/root.txt'
sudo chown root:root /root/root.txt
sudo chmod u+rx /root/root.txt
sudo chmod u-w /root/root.txt
sudo chmod go-rwx /root
```
So my final script looks like so:
```bash
sudo apt-get update

sudo apt-get install -y vsftpd
sudo apt-get install -y openssh-server 

# ssh config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.factory-defaults
sudo chmod a-w /etc/ssh/sshd_config.factory-defaults
sudo systemctl restart ssh

# ftp firewall allow
sudo ufw allow 20/tcp
sudo ufw allow 21/tcp

# ftp config
sudo cp data/our_vsftpd.conf /etc/vsftpd.conf
sudo service vsftpd restart

# ftp file serve
sudo mkdir -p /var/ftp/pub
sudo chown nobody:nogroup /var/ftp/pub
echo "john:P@ssw0rd" | sudo tee /var/ftp/pub/creds.txt


sudo bash -c 'echo "flag{i_hope_this_worked}" > /home/john/user.txt'
sudo chown john:john /home/john/user.txt
sudo chmod u+rx /home/john/user.txt
sudo chmod u-w /home/john/user.txt
sudo chmod go-rwx /home/john

sudo bash -c 'echo "flag{root_flag_poggers}" > /root/root.txt'
sudo chown root:root /root/root.txt
sudo chmod u+rx /root/root.txt
sudo chmod u-w /root/root.txt
sudo chmod go-rwx /root
```
Now just so you know you don't have to have a script. You can very easily do a few commands manually and place the root flag with permissions manually(once its there it doesn't move unless you delete it).
But i prefer to do things like this, I advise you to do the same, it definitely saved me time due to me being stupid and not knowing what to do next.

We ran this script and then shut down our vm.
I then exported the vm and uploaded it to tryhackme, I set it as a private room just because this was purely a test to see how to make a box and I'm happy it worked first time!

If for some reason you want to play the machine the link is here [tryhackme machine](tryhackme.com/jr/oopsie)

Thanks for reading hope you enjoyed :)

Thanks to John Hammond and Jambot3000 and their posts on how to make machines and making the concept clearer
