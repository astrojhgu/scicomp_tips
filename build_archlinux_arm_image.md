[ref](http://graznik.de/posts/emulate-raspberry-pi-with-qemu/)

# Preparation
We need a proper kernel and the Arch Linux ARM distro for the Raspberry Pi system. You can get both using wget:
```
wget https://github.com/dhruvvyas90/qemu-rpi-kernel/raw/master/kernel-qemu-4.4.34-jessie
wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-latest.tar.gz
```
Check the [qemu-rpi-kernel github page](https://github.com/dhruvvyas90/qemu-rpi-kernel/) for a more recent kernel if needed.

# Image creation
Choose an appropriate size for your image, I'll go for 8 GiB in this example:
```
dd if=/dev/zero of=Arch.img count=8192 bs=1048576
```

We create two partitions inside the image, one 100 MiB partition for /boot and one / partition filling the rest of the image:
```
fdisk ./Arch.img
  Command (m for help): n
    Partition type
      p   primary (0 primary, 0 extended, 4 free)
      e   extended (container for logical partitions)
    Select (default p): p
    Partition number (1-4, default 1):
    First sector (2048-16777215, default 2048):
    Last sector, +sectors or +size{K,M,G,T,P} (2048-16777215, default 16777215): +100M

    Created a new partition 1 of type 'Linux' and of size 100 MiB.

    Command (m for help): n
    Partition type
      p   primary (1 primary, 0 extended, 3 free)
      e   extended (container for logical partitions)
    Select (default p): p
    Partition number (2-4, default 2):
    First sector (206848-16777215, default 206848):
    Last sector, +sectors or +size{K,M,G,T,P} (206848-16777215, default 16777215):

    Created a new partition 2 of type 'Linux' and of size 7.9 GiB.

    Command (m for help): w
```

# Arch Linux ARM installation
Using (losetup)[https://linux.die.net/man/8/losetup] and (partx)[https://linux.die.net/man/8/partx], we setup a loop device for the image and add specific partitions:
```
sudo losetup -f --show Arch.img
sudo partx -a /dev/loop0
```

The partitions are now accessible as /dev/loop0p1 and /dev/loop0p2 within the local filesystem. Time to create an appropriate filesystem for each partition. We choose FAT for the /boot partition and ext4 for the / partition.
```
sudo mkfs.vfat /dev/loop0p1
sudo mkfs.ext4 /dev/loop0p2
```

We are now able to locally mount both partitions:
```
mkdir boot root
sudo mount /dev/loop0p1 boot/
sudo mount /dev/loop0p2 root/
```

The following commands install Arch Linux ARM to the image:
```
sudo bsdtar -xpf ArchLinuxARM-rpi-latest.tar.gz -C root/
sudo mv root/boot/* boot/
```

Since ArchLinuxARM-rpi-latest.tar.gz was created to fit onto a SD Card, the fstab file does not yet fit our needs. Without the following changes to fstab, the kernel would not find the partitions:
```
sudo emacs root/etc/fstab
```

```
# Static information about the filesystems.
# See fstab(5) for details.

# <file system>  <dir>   <type>  <options>      <dump>  <pass>
-/dev/mmcblk0p1  /boot   vfat    defaults        0       0
+/dev/sda1       /boot   vfat    defaults        0       0
+/dev/sdb2       /       ext4    defaults        0       0
```

Now unmount the partitions and clean up things:
```
sudo umount /dev/loop0p1
sudo umount /dev/loop0p2
sudo partx -d /dev/loop0
sudo losetup -d /dev/loop0
```

# Run the guest system
Let's finally run the Raspberry Pi image with qemu-system-arm:
```
qemu-system-arm -kernel kernel-qemu-4.4.34-jessie -cpu arm1176 -m 256 -M versatilepb -serial stdio -append "root=/dev/sda2 rootfstype=ext4 rw" -hda Arch.img -net nic,macaddr=de:ad:be:ef:ca:fe -net user,hostfwd=tcp::5555-:22
```

Note from the command above, that qemu will forward the guest's TCP port 22 (SSH) to the host TCP port 5555. Login into your guest system as root and activate the SSH server:
```
systemctl enable sshd
systemctl start sshd
```
Now you should be able to SSH from your host into your guest system:
```
ssh -p 5555 alarm@127.0.0.1
```
Note: 
user `alarm` has password `alarm` and user `root` has password `root`
