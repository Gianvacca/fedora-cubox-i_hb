Fedora 20 Cubox-i4Pro Kernel
==============
This is a v3.14-rc4 kernel patched with rmk's changes for the Cubox-i. It also includes a .config for Fedora 20 and other minor changes to work on the Cubox-i4Pro

What works
--------------
- All 4 Cores and other basics
- Booting from MMC
- Serial Output
- HDMI Output
- eSata
- USB
- Sound (probably S/PDIF only without better X11 driver than fbdev)
- RTC
- Infrared
- Wireless
- Bluetooth

What does not work
--------------
- UHS for mmc0
- fbdev is working for X11, but a better driver would improve performance

Installing a bootable Fedora 20 image
--------------
Download Fedora 20 Minimal, the u-boot images, and kernel

    wget http://mirror.nexcess.net/fedora/releases/20/Images/armhfp/Fedora-Minimal-armhfp-20-1-sda.raw.xz
    wget https://github.com/jmontleon/fedora-20-cubox-i4pro-binary/blob/master/u-boot/SPL?raw=true -O SPL
    wget https://github.com/jmontleon/fedora-20-cubox-i4pro-binary/blob/master/u-boot/u-boot.img?raw=true -O u-boot.img
    wget https://github.com/jmontleon/fedora-20-cubox-i4pro-binary/blob/master/rpms/kernel-3.14.0-204.rc4.cubox_i4pro.fc20.armv7hl.rpm?raw=true -O kernel-3.14.0-204.rc4.cubox_i4pro.fc20.armv7hl.rpm
    wget https://github.com/jmontleon/fedora-20-cubox-i4pro-binary/blob/master/rpms/cubox-i-brcm4329-bluetooth-1.0-1.fc20.armv7hl.rpm?raw=true
Write everything to the media, and perform some additional setup

    xzcat Fedora-Minimal-armhfp-20-1-sda.raw.xz > /dev/<location-of-your-fedora-20-arm-media>
    dd if=SPL of=/dev/<location-of-your-fedora-20-arm-media> bs=512 seek=2
    dd if=u-boot.img of=/dev/<location-of-your-fedora-20-arm-media> bs=1K seek=42
    partprobe /dev/<location-of-your-fedora-20-arm-media>
    mkdir /mnt/f20cuboxi4root
    mount /dev/<location-of-your-fedora-20-arm-media>3 /mnt/f20cuboxi4root
    mount /dev/<location-of-your-fedora-20-arm-media>1 /mnt/f20cuboxi4root/boot
    # Depending on your card reader the previous two lines may also end up being:
    # mount /dev/<location-of-your-fedora-20-arm-media>p3 /mnt/f20cuboxi4root
    # mount /dev/<location-of-your-fedora-20-arm-media>p1 /mnt/f20cuboxi4root/boot
    rm -f /mnt/f20cuboxi4root/var/lib/rpm/__*
    rm -f /mnt/f20cuboxi4root/boot/boot.*
    unlink /mnt/f20cuboxi4root/etc/systemd/system/multi-user.target.wants/initial-setup-text.service
    sed -i s@^root:\\*:@root:\\\$6\\\$VpqypThR\\\$QZF3tM8USR6bnIK.CQn3bnj0SU5VeStkKA56ZEtAoPCECe23RqPgWzafuoKGzdWzUz9z8ctjSEhHrVg63wzra0:@g /mnt/f20cuboxi4root/etc/shadow
    rpm -i --noscripts --ignorearch --root /mnt/f20cuboxi4root ./kernel-3.14.0-204.rc4.cubox_i4pro.fc20.armv7hl.rpm
    rpm -i --noscripts --ignorearch --root /mnt/f20cuboxi4root ./cubox-i-brcm4329-bluetooth-1.0-1.fc20.armv7hl.rpm
    umount /mnt/f20cuboxi4root/boot
    umount /mnt/f20cuboxi4root
    rmdir /mnt/f20cuboxi4root

    fdisk /dev/<location-of-your-fedora-20-arm-media> <<EOF
    d
    3
    n
    p

    1251954

    w
    EOF
    e2fsck -f /dev/<location-of-your-fedora-20-arm-media>3
    resize2fs /dev/<location-of-your-fedora-20-arm-media>3
    # Depending on your card reader the previous two lines may also end up being:
    # e2fsck -f /dev/<location-of-your-fedora-20-arm-media>p3
    # resize2fs /dev/<location-of-your-fedora-20-arm-media>p3

Building your own u-boot
--------------
    git clone https://github.com/rabeeh/u-boot-imx6.git
    cd u-boot-imx6
    make mx6_cubox-i_config
    make
    sudo dd if=SPL of=/dev/<location-of-your-fedora-20-arm-media> bs=512 seek=2
    sudo dd if=u-boot.img of=/dev/<location-of-your-fedora-20-arm-media> bs=1K seek=42

Building your own kernel
--------------
You can build using the SRPM:

    yum -y install yum-utils rpm-build
    yumdownloader --source kernel-3.14.0-204.rc4.cubox_i4pro.fc20
    yum-builddep -y kernel-3.14.0-204.rc4.cubox_i4pro.fc20.src.rpm
    rpm -ivh  kernel-3.14.0-204.rc4.cubox_i4pro.fc20.src.rpm
    rpmbuild -bb ~/rpmbuild/SPECS/kernel.spec

Or you can clone the repo:

    git clone https://github.com/jmontleon/fedora-20-cubox-i4pro.git
    cd fedora-20-cubox-i4pro
    make oldconfig #Or make menuconfig if you want to change stuff, and so on.
    make zImage
    make dtbs
    sudo cp arch/arm/boot/zImage /boot
    sudo cp arch/arm/boot/dts/imx6dl-hummingboard.dtb /boot
    sudo cp System.map /boot
    sudo su -
    cd /boot
    cat zImage imx6dl-hummingboard.dtb > zImage_dtb;  mkimage -A arm -O linux -T kernel -a 0x10008000 -e 0x10008000 -n vmlinuz-3.14.0 -C none -d zImage_dtb uImage
    exit
    make modules
    sudo make modules_install

Ensure you have a uEnv.txt if you haven't already:

    bootfile=/uImage
    mmcargs=setenv bootargs root=/dev/mmcblk0p3 rootfstype=ext4 rootwait console=ttymxc0,115200n8 console=tty1 video=mxcfb0:dev=hdmi,1920x1080M@60,if=RGB24,bpp=32


