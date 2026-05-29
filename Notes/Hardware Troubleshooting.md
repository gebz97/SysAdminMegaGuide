#hardware #troubleshooting #linux 
## System Info
```bash
uname -a                  # kernel + arch
lscpu                     # CPU details
lsmem                     # memory layout
dmidecode                 # full DMI/BIOS hardware info (run as root)
inxi -Fxz                 # full system summary (install if missing)
```
## Disks & Storage
```bash
lsblk -f                  # block devices + filesystems
fdisk -l                  # partition tables
blkid                     # UUIDs + fs types
smartctl -a /dev/sdX      # SMART health data (smartmontools)
hdparm -tT /dev/sdX       # disk read speed test
dmesg | grep -i "error\|fail\|ata\|nvme"   # disk errors in kernel log
```
## Memory
```bash
free -h                   # RAM + swap usage
vmstat -s                 # memory stats
memtester 512M 1          # basic RAM test (run as root)
# For thorough testing: boot Memtest86+
```
## GPU
```bash
lspci | grep -i vga       # detect GPU
glxinfo | grep renderer   # OpenGL renderer
nvidia-smi                # NVIDIA status
radeontop                 # AMD GPU usage
dmesg | grep -i drm       # GPU kernel messages
```
## Network Hardware
```bash
ip link                   # interfaces + state
lspci | grep -i net       # PCI network cards
lsusb | grep -i net       # USB network adapters
ethtool eth0              # NIC details + link status
dmesg | grep -i firmware  # firmware load failures
```
## USB & PCI
```bash
lsusb -v                  # USB devices (verbose)
lspci -v                  # PCI devices (verbose)
lspci -k                  # which kernel driver is bound
usb-devices               # detailed USB tree
```
## Kernel & Driver Logs
```bash
dmesg -T --level=err,warn          # errors/warnings with timestamps
journalctl -k -p err               # kernel errors via systemd
journalctl -b -1 -p err            # errors from last boot
```
## Thermal & Power
```bash
sensors                   # CPU/MB temps (lm-sensors)
sensors-detect            # detect sensor chips
powertop                  # power usage per device
acpi -V                   # battery + thermal + AC info
cat /sys/class/thermal/thermal_zone*/temp   # raw thermal zones
```
## Hardware Detection / Binding
```bash
lshw                      # full hardware list
lshw -short               # summary table
lshw -class network       # filter by class
modinfo <module>          # info about a kernel module
lsmod                     # loaded modules
modprobe <module>         # load a module
rmmod <module>            # unload a module
echo "options <mod> param=val" >> /etc/modprobe.d/fix.conf   # persist module options
```
## Quick Diagnostic Flow

| Symptom                | First Commands                                 |
| ---------------------- | ---------------------------------------------- |
| No display             | `dmesg \| grep drm`, `lspci \| grep VGA`       |
| Disk not found         | `lsblk`, `dmesg \| grep ata`, `smartctl`       |
| Network down           | `ip link`, `dmesg \| grep firmware`, `ethtool` |
| System overheating     | `sensors`, `cat /sys/class/thermal/...`        |
| USB device ignored     | `lsusb`, `dmesg \| grep usb`, `dmesg -T`       |
| Driver not loading     | `lsmod`, `dmesg \| grep <device>`, `lshw -k`   |
| Random crashes/freezes | `journalctl -b -1 -p err`, `memtester`         |
