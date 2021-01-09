# cryptsetup-cheat-sheet

<br><br>

# Install


## Fedora
```bash
sudo dnf install cryptsetup-luks
```


<br><br>


## Benchmark (see all available encryption algorithm)
```bash
cryptsetup benchmark
```


<br><br>


## Encrypt Partition
```bash
sudo cryptsetup --use-random -h sha512 -s 512 -c serpent-xts-plain64 -y -v luksFormat /your/partition
```
