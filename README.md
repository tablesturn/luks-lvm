# luks-lvm
How to create a safe LUKS container in linux (ubuntu) with LVM
Includes overwriting the partition with random data before creating the actual container.
For GUI fans, a lot of those commands can be performed using the gnome-disk-utility

Replace /dev/sdXY with your device path
Replace sdXY_crypt with your desired cryptconatiner name. For ease of use, don't use dashes or other special characters.
Replace the swap size with about 125% of your RAM size.

1. Create temporary encrypted partition
```shell
cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/sdXY
cryptsetup luksOpen /dev/sdXY crypttemp
```

2. Write zeros into the encrypted volume (which will turn into random data on the hdd)
```shell
dd if=/dev/zero of=/dev/mapper/crypttemp bs=256k
```

3. Close encrypted partition and overwrite header with random data
```shell
cryptsetup luksClose crypttemp
dd if=/dev/urandom of=/dev/sdXY bs=512 count=20480
```

4. Create and open new (actual) encrypted partition
```shell
cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/sdXY
cryptsetup luksOpen /dev/sdXY sdXYcrypt
```

5. Create physical volumes in the encrypted container
```shell
pvcreate /dev/mapper/sdXYcrypt
```

6. Create volume group in the new pv
```shell
vgcreate vg00 /dev/mapper/sdXYcrypt
```

7. Create logical volumes in the new vg
```shell
lvcreate -n linuxswap -L 20G vgname
lvcreate -n linuxroot -l +100%FREE vgname
```

Done.

ToDo: Make this really beautiful
