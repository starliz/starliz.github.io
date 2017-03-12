---
layout: posts
title: "Assignment 1, Xubuntu live USB"
date: 2017-01-23
---
#### Linux Basics (dat8tf063-29)

#### HAAGA-HELIA University of Applied Sciences.

#### Updated on 12.03.2017

### Some background:

I took the course "Linux basics" at Haaga-Helia UAS to boost my confidence as a Linux professional,
to refresh memory traces about things I used to do before but partially forgot. 
And to fight [impostor syndrome](https://en.wikipedia.org/wiki/Impostor_syndrome) - 
I don't have formal ICT education, I'm biologist, so I take technical courses to support skills 
and knowledge which I got working in ICT.

### The task:

Full description of the task is on [terokarvinen.com](http://terokarvinen.com/2017/agenda-for-linux-basics-dat8tf063-29-spring-2017#comment-22110)

Basically it is to create a Xubuntu live USB and try it on a computer outside the lab.

### Solution for homeworks posting.

I didn't feel creating Wordparess website useful or interesting for me, instead I decided to see how to create web pages on GitHub.

I started new repository GitHub to publish this report. [How to create a repository on Github](https://help.github.com/articles/create-a-repo/)

First I used HTTPS:

![h1-011-git-https.png]({{ site.url }}/assets/h1-011-git-https.png)

I cloned it locally:

````
[liz@localhost src]$ git clone https://github.com/starliz/hh_linux_basics.git
[liz@localhost src]$ ls
hh_linux_basics  hh_qa_course
[liz@localhost src]$ git config --global user.name "[Lidia Zalevskaya]"
[liz@localhost src]$ git config --global user.email "[zalevskaya.lidia@yandex.ru]"
[liz@localhost src]$ cd ./hh_linux_basics/
[liz@localhost hh_linux_basics]$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   h1_live_usb.html
[liz@localhost hh_linux_basics]$ git add h1_live_usb.html 
[liz@localhost hh_linux_basics]$ git commit -m "[h1] added git cheat sheet url"
[master f25f355] [h1] added git cheat sheet url
 1 file changed, 9 insertions(+), 2 deletions(-)
[liz@localhost hh_linux_basics]$ git push origin master
````

Initially it was just html file in git repo, then I changed it to PDF, and later came to GitHub pages form (using markdown and jekyll).

Made a commit and tried push it to GitHub, but then I got login request and recalled that there is a way to get rid of it. 

![h1-012-git-clone-ssh.png]({{ site.url }}/assets/h1-012-git-clone-ssh.png)

This is how I got it working: 
````
[liz@localhost hh_linux_basics]$ git remote set-url origin git@github.com:starliz/hh_linux_basics.git
````

Take public key (from local file ~/.ssh/id_rsa.pub ) and give it to GitHub (Profile/Settings/SSH and GPG keys/).
[How to add a new ssh key to your GitHub account.](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)

After this I could push from my local repo to origin without giving login and password each time.
````
[liz@localhost hh_linux_basics]$ git push origin master -u
````

And I used gitk to view git tree: 

![h1-02-gitk.png]({{ site.url }}/assets/h1-02-gitk.png)


### Creating USB stick itself:

The image can be found on [ubuntu.com](http://se.archive.ubuntu.com/mirror/cdimage.ubuntu.com/xubuntu/releases/16.04/release/), selecting ``xubuntu-16.04.1-desktop-amd64.iso``.

I used ``mediawriter`` utility as described in  [How to create and use Live USB](https://fedoraproject.org/wiki/How_to_create_and_use_Live_USB):

````
[liz@localhost hh_linux_basics]$ sudo dnf install mediawriter
````

I run the tool, gave it image location and USB device to write to, it finished successfully.

![h1-03-mediawriter.png]({{ site.url }}/assets/h1-03-mediawriter.png)

Took old notebook Sony VAIO.

Made BACKUP, BACKUP, BACKUP, BACKUP!!!

Shut it down.

Plugged in USB with Xubuntu, power on.

Image loaded from USB automatically, there was no need to select a medium to boot from.

![h1-04-ubuntu.jpg]({{ site.url }}/assets/h1-04-ubuntu.jpg)

Set up wi-fi connection:

![h1-05-wifi.jpg]({{ site.url }}/assets/h1-05-wifi.jpg)

Pinged ya.ru (all packets sent and received):

![h1-06-ping.jpg]({{ site.url }}/assets/h1-06-ping.jpg)

Opened Wikipedia in Browser, run ‘sudo lshw -short -sanitize’ in terminal.

And here is the output:

````
xubuntu@xubuntu:~$ sudo lshw -short -sanitize
H/W path                 Device      Class          Description
===============================================================
                                     system         VGN-TZ3RMN_N (N/A)
/0                                   bus            VAIO
/0/0                                 memory         111KiB BIOS
/0/4                                 processor      Intel(R) Core(TM)2 Duo CPU     U7600  @ 1.20GHz
/0/4/5                               memory         32KiB L1 cache
/0/4/6                               memory         2MiB L2 cache
/0/a                                 memory         2GiB System Memory
/0/a/0                               memory         2GiB SODIMM DDR
/0/a/1                               memory         SODIMM [empty]
/0/100                               bridge         Mobile 945GM/PM/GMS, 943/940GML and 945GT Express Memory Controller Hub
/0/100/2                             display        Mobile 945GM/GMS, 943/940GML Express Integrated Graphics Controller
/0/100/2.1                           display        Mobile 945GM/GMS/GME, 943/940GML Express Integrated Graphics Controller
/0/100/1b                            multimedia     NM10/ICH7 Family High Definition Audio Controller
/0/100/1c                            bridge         NM10/ICH7 Family PCI Express Port 1
/0/100/1c/0              enp2s0      network        88E8055 PCI-E Gigabit Ethernet Controller
/0/100/1c.1                          bridge         NM10/ICH7 Family PCI Express Port 2
/0/100/1c.1/0            wlp3s0      network        PRO/Wireless 3945ABG [Golan] Network Connection
/0/100/1c.2                          bridge         NM10/ICH7 Family PCI Express Port 3
/0/100/1c.3                          bridge         NM10/ICH7 Family PCI Express Port 4
/0/100/1d                            bus            NM10/ICH7 Family USB UHCI Controller #1
/0/100/1d/1              usb2        bus            UHCI Host Controller
/0/100/1d.1                          bus            NM10/ICH7 Family USB UHCI Controller #2
/0/100/1d.1/1            usb3        bus            UHCI Host Controller
/0/100/1d.2                          bus            NM10/ICH7 Family USB UHCI Controller #3
/0/100/1d.2/1            usb4        bus            UHCI Host Controller
/0/100/1d.3                          bus            NM10/ICH7 Family USB UHCI Controller #4
/0/100/1d.3/1            usb5        bus            UHCI Host Controller
/0/100/1d.7                          bus            NM10/ICH7 Family USB2 EHCI Controller
/0/100/1d.7/1            usb1        bus            EHCI Host Controller
/0/100/1d.7/1/2          scsi2       storage        Cruzer Blade
/0/100/1d.7/1/2/0.0.0    /dev/sdb    disk           8004MB SCSI Disk
/0/100/1d.7/1/2/0.0.0/2  /dev/sdb2   volume         15EiB Windows FAT volume
/0/100/1d.7/1/5          scsi3       storage        Sony VAIO USB Internal Optical Drive
/0/100/1d.7/1/5/0.0.0    /dev/cdrom  disk           DVD-RAM UJ862PS
/0/100/1d.7/1/6                      bus            HighSpeed Hub
/0/100/1d.7/1/6/2                    communication  BCM2046 Bluetooth Device
/0/100/1d.7/1/8                      multimedia     Visual Communication Camera VGP-VCC7 [R5U870]
/0/100/1e                            bridge         82801 Mobile PCI Bridge
/0/100/1e/4                          bridge         RL5c476 II
/0/100/1e/4.1                        bus            R5C832 IEEE 1394 Controller
/0/100/1e/4.4                        generic        R5C592 Memory Stick Bus Host Adapter
/0/100/1f                            bridge         82801GBM (ICH7-M) LPC Interface Bridge
/0/100/1f.1                          storage        82801G (ICH7 Family) IDE Controller
/0/100/1f.3                          bus            NM10/ICH7 Family SMBus Controller
/0/1                     scsi0       storage        
/0/1/0.0.0               /dev/sda    disk           100GB TOSHIBA MK1011GA
/0/1/0.0.0/1             /dev/sda1   volume         200MiB Linux filesystem partition
/0/1/0.0.0/2             /dev/sda2   volume         850MiB Linux LVM Physical Volume partition
/0/1/0.0.0/3             /dev/sda3   volume         92GiB Extended partition
/0/1/0.0.0/3/5           /dev/sda5   volume         1953MiB Linux swap / Solaris partition
/0/1/0.0.0/3/6           /dev/sda6   volume         90GiB Linux LVM Physical Volume partition
````

### MACHINOTERO on my own laptop (Lenovo x230/Fedora 24):

Machinotero source: [http://terokarvinen.com/machinotero](http://terokarvinen.com/machinotero)

Terminal output:

````
[liz@localhost ~]$ cd ./bin/
[liz@localhost bin]$ mcedit machinotero.sh                            ← To paste code there
[liz@localhost bin]$ ls -al machinotero.sh 
-rw-rw-r--. 1 liz liz 2699 Jan 18 14:56 machinotero.sh
[liz@localhost bin]$ chmod +x machinotero.sh 
[liz@localhost bin]$ ls -al machinotero.sh 
-rwxrwxr-x. 1 liz liz 2699 Jan 18 14:56 machinotero.sh
[liz@localhost bin]$ sudo bash ~/bin/machinotero.sh
uname -a; cat /etc/lsb-release; cat /var/lib/acpi-support/*-*; free -m; df -h; lshw; ifconfig; iwconfig; 
ip link; ip addr; ip route; ip rule; ip neigh; ip tunnel; ip maddr; ip mroute; dpkg --list linux-image* xserver-xorg hal; 
grep -v ^# /etc/apt/sources.list; cat /etc/apt/sources.list.d/*; dpkg --list xresprobe dmidecode lshw vbetool smartmontools; 
tail /var/log/apache/error.log; tail /var/log/apache2/error.log; lsmod; dmidecode; lspci; lspci -n; lspci -v; lsusb; fdisk -l; 
mount; cat /etc/fstab; grep /dev/sda /etc/fstab; smartctl -a /dev/sda; tune2fs -l /dev/sda1; tune2fs -l /dev/sda2; 
grep /dev/sda /var/log/syslog; ddcprobe; vbetool vbefp panelsize; dmesg; dmesg; head -25 /var/log/syslog; 
tail -25 /var/log/syslog; cat /etc/X11/xorg.conf; cat /var/log/Xorg.0.log; lsusb -v; 
packages *driver* n9-usb-copy-home.sh usb-nat-enable *grub* *firefox* *thunderbird* 915resolution*; 
Done.
````

It seems to me, it should run commands and return stderr and stdout, output should have headings and sections, print disk info, print package versions.
But obviously there is something wrong and I see only list of commands without desired output. 
TODO: check what's wrong.


History of modifications can be found in [Git repository: github.com/starliz/starliz.github.io](https://github.com/starliz/starliz.github.io/commits/master).


* * *

### Used articles:

[How to create a repository on Github](https://help.github.com/articles/create-a-repo/)

[How to add a new ssh key to your GitHub account.](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)

[How to create and use Live USB](https://fedoraproject.org/wiki/How_to_create_and_use_Live_USB)

##### “ Copyright 2017 Lidia Zalevskaya, GNU General Public License, version 3 or later.”
