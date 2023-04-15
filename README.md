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
yes | mkfs.fat /dev/nvme0n1p1 && yes | mkfs.ext4 -L ROOT /dev/nvme0n1p2 && yes| mkfs.ext4 -L HOME /dev/nvme0n1p3 && yes | mkfs.ext4 -L DATA /dev/nvme0n1p4 && fatlabel /dev/nvme0n1p1 BOOT
```

## mount partitions
```
mount /dev/nvme0n1p2 /mnt && mkdir /mnt/{boot,home,data,Virtualization,Storage} && mount /dev/nvme0n1p1 /mnt/boot && mount /dev/nvme0n1p3 /mnt/home && mount /dev/nvme0n1p4 /mnt/data && mount /dev/nvme1n1 /mnt/Virtualization/ && mount /dev/sdc /mnt/Storage
```

## set ntp
```
timedatectl set-ntp true
```

## change mirrors
```
reflector --country Germany,Austria --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
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
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && echo "KEYMAP=us" >> /etc/vconsole.conf && echo "LANG=en_US.UTF-8" >> /etc/locale.conf && locale-gen && echo "LANG=en_US.UTF-8" >> /etc/locale.conf
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

## enable wheel
```
sed -i "s/# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/g" /etc/sudoers
```

## setup multilib
```
sed -i "/\[multilib\]/,/Include/"'s/^#//' /etc/pacman.conf
```
