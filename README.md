# Pre-requisistes

1) You'll need a pc with:</br>
   - uefi enabled
   - secure boot enabled
   - tpm enabled</br>
 
2) Set in the bios </br>
   - power-on passowrd
   - bios password</br>

They can be easily bypassed (see bios-pw.org) but if someone will bypass them with a master key, at the next boot you will notice the absence of the prompt that ask for the password.

3) Linux installation with encrypted /.</br>

# Considerations</br>
On a classic setup, or the one above we have two unencrypted partitions, /boot and /efi.</br>
My workarounds to detect the evil maid attack.</br>

  1) The shasum of the partitions dump change every reboot, so I've decided to check the sha512 of every file in the /boot and /efi partition and compare them every startup. Note -> if you start windows, the script will detect the changes in the windows entries, I personally want it but if you don't; change the first check to exclude the Microsoft folder inside the /efi prtition. </br>
</br>
    With that, if someone will do an evil maid attack we'll notice it and since the script is configured to not power on internet unitil all checks are passed, the grabbed password will not be transmitted. There're other things to consider.... Two at least: uefi malware and ssd or hdd with a malware in the firmware. So here's my effort trying to mitigate the impossible.
</br>
</br>

   - extract ssd infos like Model Number, Serial Number, Firmware Revision etc... and compare them every startup.
   - extract bios infos like Version, Release Date, Runtime Size, ROM Size etc... and compare them every startup.
   - extract the hashes from pcr_0 (tpm) that is where bios and extensions are located and compare them every startup.
   - check the tpm slot that have the initrd hash and compare it every startup.

# Extras
This script do other things too:
  - update the system
  - change the mac address 
  - disable internet at boot and enable it only when all checks are passed</br>
  
 The commands that update the system and install the dependencies are for debian based distros, if you have something else replace them with your package manager.

# Running
Download the script. Open it and change the varibales as you need.</br>

      efipartition=/dev/sda? # put your efi partition
      ssd=/dev/sda # put your hdd or ssd name
      ethernet=eth0 # put your network ethernet interface name 
      wireless=wlan0 # put your wireless interface name
      
If you want the old eth0 and wlan0 type on terminal:</br>

       sudo sed -i 's/^GRUB_CMDLINE_LINUX_DEFAULT="/&net.ifnames=0 biosdevname=0 /' /etc/default/grub && sudo update-grub 

If you have changed the network inerfaces with the command above please reboot before run the script.</br>
When it start, it install all the dependencies and create the hashes.</br>
Configure it to run on startup (on gnome put the check.desktop inside  ~/.config/autostart and change the path of check.sh) Enjoy.

# This is my currently setup not a guide, feel free to follow or not. Dual boot windows & linux both encrypted.

# Install Windows</br>
- we need to create two partitions, one for efi and one for c.
- during the install, when you'll arrive at the partitioning menu delete all partitions and make all the space unallocated
- press shift and f10
- on cmd type: diskpart
- on diskpart type: list disk (to see all disks)
- on diskpart type: sel disk (the one where you want to install windows)
- on diskpart type : convert gpt (########### if you have mbr partition table ###########)
- on diskpart type: create partition efi size=500 
- on diskpart type: create partition primary size=xx the remaining disk size in mb
- close the cmd, select the c drive and tap install

# Post install </br>
This is a prsonal thing that I do, I don't like to have a windows recovery partition, instead I use a bootable usb if I need it.
Your choiche to follow or not. Less unencrypted partition we have, less are the probabilities to have problems.
- open the disks utility, there's will be a third partition
- open diskpart as admin
- on diskpart type: list disk (to see all disks)
- on diskpart type: select disk x (where x is the number of the disk where windows is installed)
- on diskpart type: list partition (to deect wich one is the third created)
- on diskpart type: select partition x (where x is the number of the recovery partition grabbed from the command list partition)
- on diskpart type: delete partition override
- now if all went well you'll remain with only two partitions, c and efi
- install veracrypt, chose encrypt the windows system partition and complete the setup

# Install Linux</br>
- boot from the usb your distro and chose install 
- on partition menu : 
  -  assign the previously created /efi partition as efi 
  -  create an ext4 partition of 1gb and select as mount point /boot
  -  create an ext4 partition with / as mount point and configure it as a volume for physical encryption.
Personally I don't use a swap partition, instead I use a swap file. Your choiche. 

# Post install</br>
- install grub-customizer
- open the windows entry and edit it: change /EFI/Microsoft/Boot/bootmgfw.efi to /EFI/VeraCrypt/DcsBoot.efi
- save the configuration and exit, at reboot you'll be able to choose linux or windows
- on terminal type: sudo sed -i 's/^GRUB_CMDLINE_LINUX_DEFAULT="/&net.ifnames=0 biosdevname=0 /' /etc/default/grub  && sudo update-grub

# Know bugs
On windows, sometimes the search stop working when it boot (problem related to veracrypt), to fix that error run on an elevated cmd :

  - net stop wsearch /y
  - net start wsearch 

Or make a bat file an run it as admin everytime you need.

# Credits for tpm dump
https://github.com/9elements/tpmtool for the extraction of the tpm hashes
