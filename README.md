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

# Benchmark (see all available encryption algorithm)
```bash
cryptsetup benchmark
```













<br><br>
__________________________________________________
<br><br>

# Encrypt Partition
```bash
sudo cryptsetup --use-random -h sha512 -s 512 -c serpent-xts-plain64 -y -v luksFormat /your/partition
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
