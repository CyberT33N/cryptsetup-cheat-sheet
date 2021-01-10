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

