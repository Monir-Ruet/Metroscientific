# Minimal Linux With Linux kernel , Busybox , Qemu Arm64 with different features.



## Kernel

To build the kernel Image you see my another doc.

## Busybox

##### Download

```bash
cd ..
wget https://busybox.net/downloads/busybox-1.35.0.tar.bz2
tar -xjf busybox-1.35.0.tar.bz2
cd busybox-1.35.0
```

##### Compile Busybox

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

Remember to mark Build static binary ( no shared libs) . Save and Exit.

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)
```

## Building the root filesystem

First we create the rootfs directory. Goto home directory.

```bash
cd ..
mkdir rootfs
```

Now we install BusyBox.

```bash
cd busybox-1.35.0
make ARCH=aarch64 CROSS_COMPILE=aarch64-linux-gnu- install CONFIG_PREFIX=../rootfs
cd ..
cd rootfs
mkdir proc sys dev etc etc/init.d usr/lib home home/root root
```

Finally, create an empty file etc/init.d/rcS, make it executable:

```bash
touch etc/init.d/rcS
chmod +x etc/init.d/rcS
```

Edit this rcS file

```bash
vi etc/init.d/rcS
```

Add this line

```bash
#!bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
echo /sbin/mdev > /proc/sys/kernel/hotplug
/sbin/mdev -s
ip addr add 10.0.2.15/24 dev eth0
ip link set dev eth0 up
ip route add default via 10.0.2.2 dev eth0

```

Creating a filesystem disk

```bash
cd ..
dd if=/dev/zero of=disk.img bs=1M count=100
mkfs.ext4 disk.img
mkdir -p tmpfs
sudo mount -t ext4 disk.img tmpfs/ -o loop
sudo cp -r rootfs/* tmpfs/
sudo sync
sudo umount tmpfs
rmdir tmpfs
```



Now Copy the kernel Image in this directory.

## Qemu Running...

```bash
qemu-system-aarch64 -smp 2 -M virt -m 1024 -cpu cortex-a53 -kernel Image -append 'root=/dev/vda rw console=ttyAMA0' -drive if=none,file=disk.img,id=hd0 -device virtio-blk-device,drive=hd0  -netdev user,id=eth0 -device virtio-net-pci,netdev=eth0 -nographic
```


