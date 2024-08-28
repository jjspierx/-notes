##### Download Arch install ISO
https://archlinux.org/download/

##### Format and Mount Arch Linux ISO to  USB flash
```sh
> df  #To see list of drives and get device name of USB drive
> sudo mkfs.ntfs /dev/[device_name]  #Formats flash drive
``


##### Mount ISO to USB
```sh
> sudo dd bs=4M if=Downloads/archxxx.iso of=/dev/[xxx] conv=fdatasync status=progress
```

##### Boot off flash drive (does not support secure boot, need to disable secure boot in bios if enabled

##### Setup console keyboard and fonts (optional)
```sh
> localectl list-keymaps #List console keyboard layouts
> loadkeys us #Load us layout (this is prob default anyway)
> setfont [font-name] #This is optional, otherwise use default font
```


##### Connect to WiFi
```sh
> iwctl device list #Get list of devices
> iwctl station DEVICE scan 
> iwctl station DEVICE get-networks #List available networks
> iwctl --passphrase=****** station [DEVICE] connect [SSID] #Connect to SSID
``


##### Update the system clock
```sh
> timedatectl
```

##### Partition Disks
```sh
> fdisk -l #List disk devices
>fdisk /dev/nvme0n1 #Use actual device name
```
 
##### Create 3 partitions
1. **EFI System** - Partition ~250M (prob already exists)
2. **Linux Swap** - (at least 4G, I did 16gb )
3. **Linux Root (x86-64)** -  Use remainder of drive

##### Format Partitions
```sh
> mkfs.ext4 /dev/nvme0n1p3 #Root partition
> mkswap /dev/nvme0n1p2 #Initialize swap partition
> mkfs.fat -F 32 /dev/nvme0n1p1 #EFI boot partition
```

##### Mount Partitions
```sh
> mount /dev/nvme0n1p3 /mnt #Mount root partition
> mount --mkdir /dev/nvme0n1p1 /mnt/boot # Mount boot partition
```

##### Enable swap partition
```sh
> swapon /dev/nvme0n1p2 
```

##### Install Essential Packages
```sh
> pacstrap -K /mnt base linux linux-firmware
```

##### Generate initial fstab config file
*genfstab* is a [Bash](https://wiki.archlinux.org/title/Bash "Bash") script that is used to automatically detect all mounts under a given mountpoint, its output can then be redirected into a file, usually `/etc/fstab`.
```sh
> genfstab -U /mnt >> /mnt/etc/fstab
```

##### Chroot into new system (change root directory)
```sh
> arch-chroot /mnt
```

##### Set Time, localization, hostname
```sh
> ln -sf /usr/share/zoneinfo/Region/City /etc/localtime #Set time zone
> hwclock --systohc #Set the Hardware Clock from the System Clock, and update the timestamps in _/etc/adjtime_.
> locale-gen #Generates locales
> echo "LANG=en_US.UTF-8" > /etc/locale.conf #Add LANG to local.conf
```

##### Network configuration
```sh
> echo "yourhostname" > /etc/hostname #Add yourhostname to hostname
> pacman -S networkmanager # Install network manager
```

##### Set root password
```sh
> passwd
```

##### Install bootloader (grub)
*GRUB_ is the boot loader while _efibootmgr_ is used by the GRUB installation script to write boot entries to NVRAM.*
```sh
> pacman -S grub efibootmgr  bootloader
> grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

##### Generate default grub configuration
```sh
>  grub-mkconfig -o /boot/grub/grub.cfg #Generates /boot/grub/grub.cfg
```

##### Install some other necessary packages while we are at it
```sh
> pacman -S sudo which
```

##### Reboot
```sh
> exit
> reboot
```

##### If all went well, you will boot into Arch
##### Start/Enable NetworkManager (nmcli)
```sh
> systemctl start NetworkManager.service
> systemctl enable NetworkManager.service
```


##### Connect to WiFi
```sh
> nmcli device #Get list of network devices
> nmcli device wifi list
> nmcli device wifi connect [SSID_or_BSSID] password [PASSWORD]
```


###### At this point you should have a working arch installation that will auto-connect to WiFi on boot.

##### Create non-root user and set password
```sh
> useradd -m [username]
> passwd [username]
```


##### Add some security
#####Loading Microcode (provides updates for bug-fixes that can be critical to system stability

```
> pacman -S intel-ucode
```
or
```sh
> pacman -S amd-ucode
```


##### Enforce a delay after failed login attempt
###### Add "auth optional pam_faildelay.so delay=4000000" to /etc/pam.d/system-login""

##### Setup GUI
```sh
> lspci | grep -e VGA #Determine graphics HW
```

##### Install video drivers (example for Intel below)
```sh
> pacman -S xf86-video-intel
```

##### Install Display Server (xorg)
```sh
> pacman -S xorg xterm xorg-xinit
> startx #To test xorg installation (optional)
```


##### Install Desktop Environment (kde shown below)
```sh
> pacman -S plasma
```

##### Install Display Manager (sddm shown below)
```sh
> sudo pacman -S sddm
```

###### Do not run this if you don't want GUI to run on boot.
```sh
> sudo systemctl enable sddm #Run startx manually to start
```

##### Start GUI
```sh
> sudo systemctl start sddm
```

##### Install, start, enable bluetooth packages
```sh
> sudo pacman -S bluez bluez-utils
> sudo systemctl start bluetooth.service
> sudo systemctl enable bluetooth.service
```


##### Install Git and required packages for using the AUR (base-devel and git)
```sh
> pacman -S --needed base-devel git
```

##### Install yay
*yay is an AUR helper.  You can use yay to install both pacman and aur packages.  I generally do not use yay, but initially, it is helpful to quickly install all packages you know you want off the bat.  AUR packages will not build and install on their own if they have dependencies on other AUR packages. You ave to install the dependencies first.  Most AUR packages are not dependent on other AUR packages, but a few are, such as proton-vpn.  This makes it a big pain to install via the AUR the traditional way.  yay helps with this.  I recommend NOT using yay after initial setup so you become familiar with git clone, and makepkg, the traditional way to download and compile packages from the AUR.*

```sh
> mkdir aur
> cd aur
> git clone https://aur.archlinux.org/yay.git #Clone yay git repo
> cd yay
> makepkg -sri #Build and install yay
```

If running KDE window manager, I like to disable kwallet.  This is optional, but if you use your own password manager, you don't need kwallet.  In order to disable kwallet, you have to install kwalletmanager.

```sh
> pacman -S kwalletmanager
```

Open kwalletmanager and disable

##### Install browser, file manger, office tools and some other helpful tools
Do Not Use *sudo* with yay
```sh
> yay -S ark brave-bin btop discord dolphon filelight fzf kate kfind ksysteminfo ksystemlog libreoffice-still neovim proton-vpn qimgv rclone signal spectacle strawberry tigervnc timeshift visual-studio-code-bin vlc zsh zsh-completions
```

##### Run time-shift and setup daily snapshots (optional)

###### Setup zsh, list shells, and make default (optional)
*I prefer zsh over bash, if you don't know or care, skip this*
```sh
> zsh #Switch to zsh
> chsh -l # list shells
> chsh -s /usr/bin/zsh #change to zsh (or any other than you want)
```


##### Setup powerlevel10k shell prompt for zsh (optional)
*powerlevel10k is a really nice and functonal shell prompt, especially for programmers*
[powerlevel10K setup instructions](https://github.com/romkatv/powerlevel10k?tab=readme-ov-files)
##### Download these fonts
[regular](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
[bold](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
[italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
[bold-italic](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)

##### Install powerlevel10k (optional)
```sh
> yay -S --noconfirm zsh-theme-powerlevel10k-git
> echo source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme
> ~/.zshrc
> p10k configure
```

##### Do not suspend on laptop lid close (optional)
```sh
> vim /etc/systemd/logind.conf #Un-comment lines about lid switch, and set to ignore
> systemctl restart systemd-logind.service
```

