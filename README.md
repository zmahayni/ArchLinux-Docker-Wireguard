
# Arch Linux Install Documentation  
## Setting up VM

1. Navigate to the Arch Linux page and download an appropriate ISO (mit.edu). 
   
2. Open up the terminal and cd into the directory where the iso file is held. Type the following command and verify the result with the checksum on the download page.
```Bash
    shasum -a 256 filename
```
   
3. Open UTM and create a VM with the iso you downloaded. Give appropriate RAM and memory. Change the architecture to x86_64 and make sure the UEFI_boot setting is on.


## Arch Setup

1. Start the VM in **UEFI** Mode. Verify the boot mode using the command (verify it says 64): 
```Bash
   cat /sys/firmware/efi/fw_platform_size
```

2. Verify internet connectivity using the command:
```Bash
   ping google.com
```

3. Find your timezone and change the clock using the commands:
```Bash
   timedatectl list-timezones
   sudo timedatectl set-timezone America/Chicago
```

4. Partition the hard drive into two parts. One for booting and one for the root. Give the the one for booting 500MB. Give the one for root the rest of the memory. Use the following commands:
```Bash
    fdisk -l
    fdisk /dev/vda
```

5. Format the root partition tot eh Ext4 file system and the boot partition to the FAT32 system using the commands:
```Bash
   mkfs.ext4 /dev/vda2
   mkfs.fat -F 32 /dev/vda1
```

6. Both file systems must also be mounted using the following commands:
```Bash
   mount /dev/vda2 /mnt
   mount --mkdir /dev/vda1 /mnt/boot
```

7. Essential packages can be installed by executing the pacstrap script:
```Bash
   pacstrap -K /mnt base linux linux-firmware
``` 

8. Generate the fstab file using the following command:
```Bash
   genfstab -U /mnt >> /mnt/etc/fstab
```

9. Change root into the newly created system. Install the following packages:
```Bash
   arch-chroot /mnt
   pacman -S vim man-db man-pages texinfo 
   pacman -S networkmanager iproute2 sudo
```

10. Enable the network manager using the command:
```Bash
   systemctl enable NetworkManager
```

11. Run the following commands to select the time zone and activate the hardware clock:
```Bash
   ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
   hwclock --systohc
```

12. Uncommment the line *en_US.UTF-8 UTF-8* in `/etc/locale.gen` using **vim**. Generate the `locale.conf` and change the **LANG** var using the commands:
```Bash
   vim /etc/locale.gen
   i #INSERT MODE AND UNCOMMENT LINE
   :wq #SAVE AND EXIT
   locale-gen
   vim /etc/locale.conf
   i #INSERT MODE
   LANG=en_US.UTF-8
   :wq #SAVE AND EXIT
```

13. Create the hostname file and insert *ArchLinux*.
```Bash
   vim /etc/hostname
   i # INSERT MODE
   ArchLinux
   :wq #SAVE AND EXIT
```

14. Set the root password using the `passwd` command.

## Boot Loader Setup

1. Install **GRUB** as the boot loader:
```Bash
   pacman -S grub efibootmgr
```

2. Create an EFI directory in the boot partition. Mount this boot partition to the EFI directory.
```Bash
   mkdir /boot/EFI
   mount /dev/vda1 /boot/EFI
```

3. Install **GRUB** to the disk using the following command:
```Bash
   grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB
```

4. Generate the **GRUB** configuration file using the following command:
```Bash
   grub-mkconfig -o /boot/grub/grub.cfg
```

5. Shutdown the VM, then navigate to the CD/DVD setting in UTM. Clear it. 
```Bash
   exit
   shutdown now
```

## Desktop Environment

1. Create a user and passwd for yourself. Edit the **sudoers** file to give yourself sudo permissions using the commands:
```Bash
   useradd -m zade
   passwd zade
   sudo vim /etc/sudoers
   i #INSERT MODE
   zade ALL=(ALL:ALL) ALL
   :wq!
```

2. Install the **LXQT** DE. Enable display and network managers and install any additional components. Reboot at the end to see the **LXQT** display:
```Bash
   sudo pacman -S --needed xorg
   sudo pacman -S --needed lxqt xdg-utils ttf-freefont sddm
   sudo pacman -S --needed libpulse libstatgrab libsysstat lm_sensors network-manager-applet oxygen-icons pavucontrol-qt
   reboot
```

## Color Coding Terminal

1. Make a copy of *.bashrc* to ensure there is a backup in case things go wrong.
```Bash
   cp .bashrc .bashrc.backup
```

2. Use vim to edit the *.bashrc* file and insert the following variables.
```Bash
   CYAN1="\[$(tput setaf 51)\]"
   RED="\[$(tput setaf 9)\]"
   RESET="\[$(tput sgr0)\]"
   PS1="${CYAN1}[\u@${RED}\h \W]${RESET}\$ "
```

## User Codi and Justin

1. Create a new user with the following command:
```Bash
   sudo useradd -m codi
```

2. Create a password for **codi** using the passwd command:
```Bash
   sudo passwd codi
```

3. Set the password to expire so that it must be changed after first login.
```Bash
   sudo passwd -e codi
```

4. Add **codi** to the **wheel** group to enable sudo permissions.
```Bash
   sudo usermod --append --groups wheel codi
```
5. Repeat this process with justin.

6. Edit the */etc/sudoers* file and uncomment the line with the wheel group.
```Bash
   sudo vim /etc/sudoers
   i #INSERT MODE
   %wheel ALL=(ALL) ALL
   :wq!
```

## ZSH Shell

1. Install the **zsh** package.
```Bash
   sudo pacman -S zsh
```

2. After the install, run the `zsh` command. A menu will pop up with many configuration options. Change the settings as you desire.

## Installing SSH

1. Install the **openssh** package.
```Bash
   sudo pacman -S openssh
```

## Adding Aliases

1. Edit the `~/.bashrc` file and add the following aliases, as well as any others you desire.
```Bash
   vim ~/.bashrc
   alias c='clear'
   alias ..='cd ..'
   alias vi=vim
   alias update='sudo pacman -Syu'
```