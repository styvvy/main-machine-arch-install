# basic arch installation
## wipe nvme0n1
```
wipefs -a /dev/nvme0n1
```

## create partitions
```
cfdisk /dev/nvme0n1
```

## format partitions and label them

```

yes | mkfs.fat /dev/nvme0n1p1 && yes | mkfs.ext4 -L ROOT /dev/nvme0n1p2 && yes| mkfs.ext4 -L HOME /dev/nvme0n1p3 && fatlabel /dev/nvme0n1p1 BOOT
```

## mount partitions
```
mount /dev/nvme0n1p2 /mnt && mkdir /mnt/{boot,home,data,Virtualization,Storage} && mount /dev/nvme0n1p1 /mnt/boot && mount /dev/nvme0n1p3 /mnt/home && mount /dev/nvme1n1 /mnt/Virtualization/ && mount /dev/sdc /mnt/Storage
```

## set ntp
```
timedatectl set-ntp true
```

## change mirrors
```
reflector --country Germany,Austria --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

## update pacman.conf for parallel downloading
```
sed -i "s/#ParallelDownloads = 5/ParallelDownloads = 15/g" /etc/pacman.conf && sudo pacman -Sy
```

## install basic system and packages
```
pacstrap /mnt base base-devel linux linux-headers linux-firmware python3 grub efibootmgr os-prober networkmanager ntp openssh neovim
```

## generate fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```

## chroot into new system
```
arch-chroot /mnt
```

## enable services
```
systemctl enable NetworkManager.service && systemctl enable sshd.service
```

## setup locale etc.
```
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && echo "KEYMAP=us" >> /etc/vconsole.conf && echo "LANG=en_US.UTF-8" >> /etc/locale.conf && locale-gen
```

## setup grub
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub && grub-mkconfig -o /boot/grub/grub.cfg
```

## setup root password
```
passwd
```

## add user stywen
```
useradd -m stywen && usermod -G wheel stywen && passwd stywen
```

### MISC
```
sed -i "s/# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/g" /etc/sudoers && echo "stywen ALL=NOPASSWD: ALL" >> /etc/sudoers && sed -i "/\[multilib\]/,/Include/"'s/^#//' /etc/pacman.conf
```
### exit and reboot
```
exit
umount -R /mnt
reboot
```


### install yay
```
sudo pacman -S git && cd /opt && sudo git clone https://aur.archlinux.org/yay-git.git && sudo chown -R $USER:$USER ./yay-git && cd yay-git && makepkg -si
```
