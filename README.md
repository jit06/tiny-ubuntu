# tiny-ubuntu
Fast boot and light version of ubuntu minimal 18.04 for odroid C

Used kernel can be found here : https://github.com/jit06/linux/tree/odroidc-3.10.y

Features :
- based on custom 3.10.y kernel with a lot of things removed (eg: bluetooth,i2c...)
- mali-fbdev and git installed
- gcc8 as default compiler
- u-boot-tools works out of the box
- cursor in console
- journald configured with volatile storage
- tmpfs limited to 20Mb for /var/log 
- rootfs mounted with noatime and discard
- rsyslog, ModemManager, pppd-dns and loadcpufreq services disabled
- quiet boot
- uhs enabled
- vpu disabled
- no more resolution change when kernel is loaded if not in 720p
- default cpu frequence to 1.7 Ghz
- CBVS default output to 480i (NTSC)

with network (DHCP) and 720p display, systemd-analyse gives : 

    Startup finished in 2.490s (kernel) + 2.916s (userspace) = 5.406s
    graphical.target reached after 2.862s in userspace
