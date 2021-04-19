# tiny-ubuntu
Fast boot and light version of ubuntu minimal 18.04 for odroid C
It comes with two flavours : one with standard console on framebuffer (tty1 on HDMI) and another without framebuffer console (eg. perfect for a kiosk mode)

Used kernel can be found here : https://github.com/jit06/linux/tree/odroidc-3.10.y

Features
--------
- based on custom 3.10.y kernel with a lot of things removed (eg: bluetooth, i2c, hardware video acceleration...)
- mali-fbdev and git installed
- gcc8 as default compiler
- u-boot-tools works out of the box (fw_setenv and fw_printenv)
- cursor in console
- journald configured with volatile storage (and logs to ttyS0 foir the no-console version)
- tmpfs limited to 20Mb for /var/log 
- rootfs mounted with noatime and discard
- rsyslog, ModemManager, pppd-dns and loadcpufreq services disabled
- quiet boot
- uhs enabled
- vpu disabled and all video acceleration disabled (only usefull for kodi and c2play)
- no more resolution change when kernel is loaded if not in 720p (faster boot)
- default cpu frequence to 1.7 Ghz
- CBVS default output to 480i (NTSC)

Boot time
---------
with network (DHCP) and 720p display with the framebuffer console version, systemd-analyse gives : 

    Startup finished in 2.490s (kernel) + 2.916s (userspace) = 5.406s
    graphical.target reached after 2.862s in userspace

Used commands
-------------
Below are the command used on odroid C1 with ubuntu 18.04 minimal to get this result.
There are by far no rocket science commands. All can be found in the hardkernel's wiki or elsewhere.
This is just the mix I used to build this image.

Before all theses commands, the system has been upgraded (update & upgrade & dist-upgrade) and the custom kernel has been added (the original one is still available in  /media/boot/uImage.old).
Note that you need to add "fw_printenv" and "fw_setenv" to /usr/bin as the latest uboot-utils package seems not to provide then anymore. 


    # few package install
    apt-get -y install git mali-fbdev u-boot-tools gcc-8 g++-8
    
    # u-boot-tools setup
    echo "/dev/mmcblk0 0x80000 0x8000" > /etc/fw_env.config

    # enable cursor in hdmi console
    echo "
    # cursor in console
    if [ ! -f ~/.terminfo.txt ]; then
        infocmp >> ~/.terminfo.txt
        sed -i -e 's/?0c/?112c/g' -e 's/?8c/?48;0;64c/g' ~/.terminfo.txt
    fi
    tic ~/.terminfo.txt > /dev/null
    tput cnorm" >> /etc/profile

    # journald storage to volatile
    sed -i "s/#Storage=auto/Storage=volatile/" /etc/systemd/journald.conf

    # tmpfs for /var/log
    rm -Rf /var/log/*
    echo "tmpfs /var/log tmpfs nodev,nosuid,noatime,size=20M 0 0" >> /etc/fstab
    
    # optimized mount option for emmc
    sed -i "s/ext4 defaults/ext4 noatime,discard,defaults/" /etc/fstab
    
    # disabled services
    systemctl disable rsyslog
    systemctl disable ModemManager
    systemctl disable pppd-dns.service
    systemctl disable loadcpufreq.service      

    # gcc8 as default
    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-8 100
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8
    update-alternatives --set gcc "/usr/bin/gcc-8"
    update-alternatives --set g++ "/usr/bin/g++-8"
