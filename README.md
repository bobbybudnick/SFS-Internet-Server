# SFS-Internet-Server
The Software Freedom Solution Internet facing server infrastructure

![](https://github.com/bobbybudnick/SFS-Internet-Server/blob/main/IMG_20220703_0513464.jpg)

Intro
-----
Because Software Freedom Solutions has a history of dealing with small computers then it is only natural that the server platform would share a touch of the mobile world being powered by EMMC and MicroSD storage technology, use of small and lower power computer boards and driving input through unorthodox methods for servers like touch and Bluetooth.

Hardware
-----
sits in a convenient place on the workstation desk beside the main keyboard  
powered from USB by the very router it is connected to  
has a touchscreen with onscreen keyboard for direct administration  
has a mini Bluetooth keyboard attached for direct administration

Security Information
-----
could use VLAN isolation with a single computer for administration  
could make use of concept of controlled pushes and pulls for file transfers  
could communicate with an obscure protocol like stunnel secured telnet  
could utilize port knocking between the server and administration computer  
could employ a variety of operating systems between hosts  
could implement virtualization and snapshotting for server instances  
could leverage nonstandard port numbers

Setup host script
-----
install programs  
removal of annoying programs  
copy xsessionrc to setup touch and display  
copy Lattepanda original bluetooth firmware to setup bluetooth keyboard and mouse  
convenience message

Setup guest script
-----
install programs  
set lsof setuid root  
setup pcap permissions  
copy apache2.conf to setup apache web server for scripting  
copy inittab to automatically login at first virtual console  
copy profile to launch iftop at login  
convenience message

Upgrade rationale from ascii to beowulf
-----
kernel to fix ethernet - check - r8152 blacklisted so install ok but not installation  
firmware to fix bluetooth - same as ethernet with 8723bs and needed firmware  
qtvirtualkeyboard availability - check  
less broken kde panel - fail - panel is still awkward with network monitor added  
fix touch response in middle - ok not perfect maybe power related or early technology

Lattepanda BIOS entry failure
-----
USB will always boot first  
system setup should be a grub option which will enter BIOS  
Windows install has an option to enter BIOS  
this one may not be able to enter BIOS with keyboard press  
on USB boot menu press e to edit any mode and f2 for console and fwsetup to enter BIOS

9c/92 error
-----
not sure this error is actually USB related  
seeing black screen after getting past 9c/92  
9c/92 and black screens started after choosing system setup and restarting from BIOS  
freezes happening at usb boot menu with drive in USB 3 port  
freezes not happening at USB boot menu with drive in USB 2 port furthest from board  
works again after grub-install and update-grub from recovery chroot  
android-ia is the first boot option for some reason  
1 way to fix may be to move the bootx64.efi file from efi partition - fixed  
1 way to fix may be to change boot order in bios - not tested  
1 way to fix may be to delete android-ia entry in efibootmgr - not tested

Hostname issues
-----
hostname stays same as old hostname after reboot  
make sure there are no spaces in hostname file

Extensive Samba failures
-----
computers of different types all fail to connect to Samba server  
create or modify smb.conf or smb4.conf and add client max protocol = SMB3

No way to launch programs
-----
quicklaunch plasma widget missing  
install plasma-widgets-addons

Black screen failures
incompatibilities between 7" screen and Lattepanda original  
screen does get in inconsistent state when the Lattepanda is shutdown  
after Lattepanda bootup there is a black screen and virtual consoles do not respond  
ideally the Lattepanda should be booted just after being powered on

Network monitor weirdness
-----
network monitor pushes clock off right side of screen when to the left of system tray  
move network monitor to right of clock - still looks kind of strange

Disable weird ethernet names
-----
previously grub boot edits were used  
now /lib/udev/rules.d/73-usb-net-by-mac.rules moved to home

Vmware notes
-----
install from a host USB drive on guest may need to be mounted which is odd so use iso  
512MB RAM per guest does start expert installer in low RAM mode but installs properly  
turn off automatic input grab so the touchscreen does not get locked in place  
Vmware tools also prevents input grab by dealing with mouse input more seamlessly  
to interact directly use a keyboard and mouse but should be able to shutdown with touch

Vmware signal 6 crash
-----
caused by Vmware and ALSA audio bug  
possibly related to emulated system boot beep  
move bugged asoundrc away to fix

Automatic console login on guest
-----
edit /etc/inittab  
add --autologin pi after agetty - works for sysvinit only
                                  
Iftop capture privileges on guest  
-----
sudo no passwords at all - not acceptable for server  
grant access to iftop only in sudo with nopasswd - not bad  
setuid iftop - not bad  
set network privileges for iftop with setcap - maybe most secure - see below  
sudo setcap cap_net_raw,cap_net_admin+eip /usr/sbin/iftop

Automatic start of iftop at login on guest
-----
/etc/profile or ~/.bash_profile or ~/.bash_login  
~/.profile - used and works well

Execute python scripts in apache on guest
-----
make sure scripts are executable and place in /usr/lib/cgi-bin  
LoadModule cgi_module /usr/lib/apache2/modules/mod_cgi.so in apache config

Cgi-bin security check
-----
the scripts here should not be readable to the outside world  
use wget to try to download scripts  
a download completes but it is only the output of the script not the contents

Python memory failures on web server
-----
a memoryerror exception is being thrown by Python  
the problem is as suspected in the read() call  
try some more memory limitation tricks in script - fail  
try to increase guest memory allocation to 1280 MB - host freezes up  
try to add swap to guest - bad idea - 2 GB swap added to 2 GB RAM host instead  
try to see about chunk splitting - could not get this to work  
ultimately guest memory allocation was set to 1 GB  
the guest goes down to around 50 MB RAM when uploading so this may be a borderline fix  
the host strangely stays around 550 MB RAM / 100 MB swap used so Vmware is pretty neat
            
No Python errors shown through Apache
-----
strange error shown in Apache error log  
Response header name '<!--' contains invalid characters  
this is because when the cgit python module crashes it throws junk characters to Apache  
this trips up Apache particularly because junk header characters can be a hack  
this has only recently within a few years started with an apache security change  
unfortunately the Python cgit module looks scheduled for removal anyway  
HttpProtocolOptions Unsafe can work around the problem in apache config
                                  
Host user inteface
-----
KDE with network monitor widget on panel  
Vmware  
Ksysguard with smallest height at bottom

Basic server preparation
-----
make MISC directory  
make SFS_MID_CODE directory  
make SFS_MID_WIKI directory  
fdisk new drive  
mkfs.ext4 new drive  
setup fstab for new drive  
copy wiki  
copy repo files  
copy guest files  
run guest script  
run web script  
change cgi security statement

Vmware meltdown
-----
the small 32 GB EMMC was filling up instead of the micro SD card as expected  
this led to both virtual machines having to be entirely deleted  
the extra virtual disk being remote from the Vmware folder is causing some problems  
it is important that a template be made with no extra virtual disks  
clone from this template before the extra virtual disk is added  
originally the template was used for troubleshooting but now it is just a backup  
Vmware is fundmentally incompatible with the 2 disparate virtual disks approach  
both virtual disks must be in the same folder and on the same drive  
can back up full virtual machine with both disks because it does not expand initially  
one way is to put both disks on host mounted SD but this would wear the SD fast  
another way is to start Vmware as root and add the extra disk as a block device  
the other way does in fact conserve space on the EMMC as it is supposed to  
the other way has fairly poor latency to the block device but bandwidth is good enough  
the other way needs physical disk set to independent in advanced for snapshots to work  
the independent mode causes snapshots to not affect the physical disk  
the independent mode even has an option to make the disk nonpersistent

