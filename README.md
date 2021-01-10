# cryptsetup-cheat-sheet

<br><br>

# Install


## Fedora
```bash
sudo dnf install cryptsetup-luks
```
















<br><br>
__________________________________________________
<br><br>

# Guides
- 
Ubuntu 20.04 btrfs-luks full disk encryption including /boot and auto-APT snapshots with Timeshift
```bash
- usi efi bootloader

- boot from cd and run "try mode"


mount | grep efivars
-> should sa efivars on /sys/firmware/efi/efivars


# cache admin session
sudo -i


# get name of your harddrive (mostly sda)
lsblk


# partition your harddrive
parted /dev/sda
mklabel gpt
# create efi partition
mkpart primary 1MiB 513MiB
# create swap partition 4GB
mkpart primary 513MiB 4609MiB
# create root file system with remaining free space
mkpart primary 4609MiB 100%
#check if it works
print
#exit
q


# install gparted to verify that it worked
dnf install gparted


# create luks partition
sudo cryptsetup --use-random -h sha512 -s 512 -c serpent-xts-plain64 -y -v luksFormat /dev/sda3


# creater mappe device
cryptsetup luksOpen /dev/sda3 cryptdata

# check if it worked
ls /dev/mapper


mkfs.btrfs /dev/mapper/cryptdata




## https://youtu.be/yRSElRlp7TQ?t=445 ##
## optional - optimise boot options - start ## 
gedit /usr/lib/partman/mount.d/70btrfs
- edit for / "subvol=@,ssd,noatime,space_cache,commit=120,compress=zstd"
- edit for /home "subvol=@home,ssd,noatime,space_cache,commit=120,compress=zstd"

gedit /usr/lib/partman/fstab.d/btrfs
- edit "pass=0"
- edit for / "subvol=@,ssd,noatime,space_cache,commit=120,compress=zstd"
- edit for /home "subvol=@home,ssd,noatime,space_cache,commit=120,compress=zstd"
- edit "$home_options" 0 0
## optional - optimise boot options - end ## 





# run your installation process
# on ubuntu
ubiquity --no-bootloader

# go to partition manager while ask where to install "something else"
- on sda1 choose "efi system partition"
- on sda2 choose "swap"
- on /dev/mapper/cryptdata btrfs choose "btrfs journaling file system"
check "format the partition"
mount point "/"

- Install Ubuntu
- After finish click "Continue Testing"


mount -o subvol=@,ssd,noatime,space_cache,commit=120,compress=zstd /dev/mapper/cryptdata /mnt

for n in proc sys dev etc/resolv.conf; do mount --rbind /$n /mnt/$n; done

chroot /mnt

# check if worked and mnt can be found
ls


# mount everything
mount -av

# create keyfile
mkdir /etc/luks
dd if=/dev/urandom of=/etc/luks/boot_os.keyfile bs=4096 count=4
chmod u=rx,go-rwx /etc/luks
chmod u=r,go-rwx /etc/luks/boot_os.keyfile
cryptsetup luksAddKey /dev/sda3 /etc/luks/boot_os.keyfile

# check if it worked
cryptsetup luksDump /dev/sda3 | grep "Key Slot"


# prevent leaking key material
nano /etc/cryptsetup-initramfs/conf-hook
- scroll to bottom of file and change:
KEYFILE_PATTERN=/etc/luks/*.keyfile

nano /etc/initramfs-tools/initramfs.conf
- scroll to bottom and add:
UMASK=0077



# check if crypttab exist
cat /etc/crypttab

# get UUID of luks partition - in our case of sda3
blkid

# create crypttab
nano /etc/crypttab
cryptdata UUID=************ /etc/luks/boot_os.keyfile luks



# create encrypted swap partition
# get UUID of swap partition - in our case of sda2
blkid
nano /etc/crypttab
cryptswap UUID=************ /dev/urandom swap,offset=1024,cipher=serpent-xts-plain64,size=512

nano /etc/fstab
- change swap UUID=****** to:
/dev/mapper/cryptswap



########## I guess optional - START ##############
# create another swap file/subvolume
mount -o subvolid=5 /dev/mapper/cryptdata /mnt
btrfs subvolume create /mnt/@swap

# make sure not another swap file is in usage
swapoff /swapfile
# if it exist run:
# rm /swapfile

# prepare the new swapfile
truncate -s 0 /mnt/@swap/swapfile
chattr +C /mnt/@swap/swapfile
btrfs property set /mnt/@swap/swapfile compression none
fallocate -l 4G /mnt/@swap/swapfile
chmod 600 /mnt/@swap/swapfile
mkswap /mnt/@swap/swapfile


# mount subvolume
mkdir /mnt/@/swap
nano /mnt/@/etc/fstab
# add the new swap file
/dev/mapper/cryptdata /swap btrfs  subvol=@swap,compress=no 0 0
/swap/swapfile none swap sw 0 0
########## I guess optional - END ##############


# unmount top volume
unmount /mnt

nano /etc/default/grub
- change GRUB_CMDLINE_LINUX_DEFAULT=""
- add GRUB_ENABLE_CRYPTODISK=y

apt install -y --reinstall grub-efi-amd64-signed linux-generic linux-headers-generic

update-initramfs -c -k all


grub-install /dev/sda
update-grub

# check if key is correctly stored
lsinitramfs /boot/initrd.img | grep "^cryptroot/keyfiles/"


exit
reboot now

```

<br><br>

- Encrypt your Hard Drive/Partition in Linux (https://www.youtube.com/watch?v=ch-wzDyo-wU)
- Auto-mount Encrypted partitions at boot (https://www.youtube.com/watch?v=dT4kvmpCJfs)
```bash
# Encrypt your Hard Drive/Partition in Linux #
sudo apt install cryptsetup

sudo cryptsetup --use-random -h sha512 -s 512 -c serpent-xts-plain64 -y -v luksFormat /dev/yourpartition

sudo cryptsetup luksOpen /dev/yourpartition yourpartition

sudo fdisk -l

sudo mkfs.ext4 /dev/mapper/yourpartition

# If you do not use system drive
sudo tune2fs -m 0 /dev/mapper/yourpartition

sudo mkdir /mnt/encrypted

sudo mount /dev/mapper/yourpartition /mnt/encrypted

sudo touch /mnt/encrypted/test.txt

sudo chown -R `whoami`:users /mnt/encrypted

sudo umount /dev/mapper/yourpartition

sudo cryptsetup luksClose yourpartition




# Auto-mount Encrypted partitions at boot #

lsblk

sudo cryptsetup luksUUID /dev/yourpartition

sudo nano /etc/crypttab
# if your system drive is not encrypted then we use none
sdb1 /dev/disk/by-uuid/your-uuid-here none luks

sudo mkdir /mnt/encrypted_yourpartition

sudo nano /etc/fstab
/dev/mapper/sdb1 /mnt/encrypted_yourpartition    ext4   defaults   0  2
```


















<br><br>
__________________________________________________
<br><br>

# Benchmark (see all available encryption algorithm)
```bash
cryptsetup benchmark
```













<br><br>
__________________________________________________
<br><br>

# Encrypt Partition
```bash
sudo cryptsetup --use-random -h sha512 -s 512 -c serpent-xts-plain64 -y -v luksFormat /dev/yourpartition
```













<br><br>
__________________________________________________
<br><br>

## Create logical device-mapper device
```bash
sudo cryptsetup luksOpen /your/encryptedpartition username
```

<br><br>


#### view mapping details
```bash
ls -l /dev/mapper/username
```


<br><br>


#### view status of mapping
```bash
sudo cryptsetup -v status usernmae
```












<br><br>
__________________________________________________
<br><br>


#### check if device has been formatted
```bash
sudo cryptsetup luksDump /your/encryptedpartition
```











<br><br>
__________________________________________________
<br><br>

# write zeros to encrypted partition
```bash
dd if=/dev/zero of=/dev/mapper/yourusername
```









<br><br>
__________________________________________________
<br><br>

# format partition
```bash
sudo mkfs.ext4 /dev/mapper/yourusername
```















<br><br>
__________________________________________________
<br><br>

# get UUID of encrypted partition
```bash
sudo cryptsetup luksUUID /etc/encryptedpartition
```

