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

## full disk with system
- Ubuntu 20.04 btrfs-luks full disk encryption including /boot and auto-APT snapshots with Timeshift (https://www.youtube.com/watch?v=yRSElRlp7TQ)
- https://mutschler.eu/linux/install-guides/ubuntu-btrfs/
```bash
# enable EFI for this guide
# use "try ubuntu" when you boot from ubuntu DVD

# create temp admin sessions
sudo -i

# create partitions
parted /dev/sda
mklabel gpt
# EFI
mkpart primary 1MiB 513MiB
# swap
mkpart primary 513MiB 4609MiB
# root
mkpart primary 4609MiB 100%
q

# create luks partition
cryptsetup --use-random -h sha512 -s 512 -c serpent-xts-plain64 -y -v luksFormat --type=luks1 /dev/sda3

# map encrypted partition to "cryptdat". You can use other name if you want
cryptsetup luksOpen /dev/sda3 cryptdata

# pre-format cryptdata
mkfs.btrfs /dev/mapper/cryptdata

# install ubuntu
ubiquity --no-bootloader
# Select /dev/sda1, press the Change button. Choose Use as ‘EFI System Partition’.
# Select /dev/sda2, press the Change button. Choose Use as ‘swap area’ to create a swap partition. We will encrypt this partition later in the crypttab.
# Select the root filesystem device for formatting (/dev/mapper/cryptdata type btrfs on top), press the Change button. Choose Use as ‘btrfs journaling filesystem’, check Format the partition and use ‘/’ as Mount point.
# Recheck everything, press the Install Now button to write the changes to the disk and hit the Continue button. Select the time zone and fill out your user name and password. If your installation is successful choose the Continue Testing option. DO NOT REBOOT!, but return to your terminal.

# Create a chroot environment and enter your system
mount -o subvol=@,ssd,noatime,space_cache,commit=120,compress=zstd /dev/mapper/cryptdata /mnt
for i in /dev /dev/pts /proc /sys /run; do sudo mount -B $i /mnt$i; done
rm /mnt/etc/resolv.conf
sudo cp /etc/resolv.conf /mnt/etc/
sudo chroot /mnt

# Now you are actually inside your system, so let’s mount all other partitions and have a look at the btrfs subvolumes:
mount -av
# /                        : ignored
# /boot/efi                : successfully mounted
# /home                    : successfully mounted
# none                     : ignored

# Create crypttab
export UUIDSDA3=$(blkid -s UUID -o value /dev/sda3)
echo "cryptdata UUID=${UUIDSDA3} none luks" >> /etc/crypttab

# create encrypted Swap partition
export SWAPUUID=$(blkid -s UUID -o value /dev/sda2)
echo "cryptswap UUID=${SWAPUUID} /dev/urandom swap,offset=1024,cipher=serpent-xts-plain64,size=512" >> /etc/crypttab
# adapt the fstab accordingly
sed -i "s|UUID=${SWAPUUID}|/dev/mapper/cryptswap|" /etc/fstab

# Add a key-file to type luks passphrase only once
mkdir /etc/luks
dd if=/dev/urandom of=/etc/luks/boot_os.keyfile bs=4096 count=1
chmod u=rx,go-rwx /etc/luks
chmod u=r,go-rwx /etc/luks/boot_os.keyfile
cryptsetup luksAddKey /dev/sda3 /etc/luks/boot_os.keyfile
cryptsetup luksDump /dev/vda3 | grep "Key Slot"
# Key Slot 0: ENABLED
# Key Slot 1: ENABLED
# Key Slot 2: DISABLED
# Key Slot 3: DISABLED
# Key Slot 4: DISABLED
# Key Slot 5: DISABLED
# Key Slot 6: DISABLED
# Key Slot 7: DISABLED
echo "KEYFILE_PATTERN=/etc/luks/*.keyfile" >> /etc/cryptsetup-initramfs/conf-hook
echo "UMASK=0077" >> /etc/initramfs-tools/initramfs.conf
sed -i "s|none|/etc/luks/boot_os.keyfile|" /etc/crypttab

# Install the EFI bootloader
echo "GRUB_ENABLE_CRYPTODISK=y" >> /etc/default/grub

apt install -y --reinstall grub-efi-amd64-signed linux-generic linux-headers-generic linux-generic-hwe-20.04 linux-headers-generic-hwe-20.04
update-initramfs -c -k all
grub-install /dev/sda
update-grub

exit
reboot now
```




<br><br>


## external hard drive & partition
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

