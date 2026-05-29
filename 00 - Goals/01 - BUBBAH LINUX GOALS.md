# Mainstream Linux Topics
## Core System Administration [[linux.basics.core]]

- User & group management
- File permissions & ownership
- Package management (DNF/YUM/RPM)
- Service management (systemd basics)
- Process management (ps, top, kill, nice)
- Disk & filesystem management (fdisk, lsblk, df, du)
- LVM basics
- Mounting & /etc/fstab
- Cron jobs & scheduling
- Log management (journalctl, /var/log)

## Networking Basics [[linux.basics.net]]

- Interface configuration (nmcli, nmtui)
- DNS resolution (/etc/resolv.conf, /etc/hosts)
- Firewall basics (firewalld, basic iptables)
- SSH configuration & key management
- Network troubleshooting (ping, traceroute, ss, netstat, tcpdump basics)

## Security Basics [[linux.basics.security]]

- sudoers configuration
- SELinux basics (modes, contexts, booleans, audit2allow)
- Basic PAM configuration
- Password policies
- SSH hardening
- File integrity basics

## Storage Basics [[linux.basics.storage]]

- Partition management
- Filesystem creation & mounting
- NFS basics
- Swap management

## Performance Basics [[linux.basics.perf]]

- top, htop, vmstat, iostat, sar
- Basic bottleneck identification (CPU, RAM, disk, network)
- /proc basics

## Shell & Scripting [[linux.basics.shell]]

- Bash scripting
- Text processing (awk, sed, grep, cut, sort)
- Piping & redirection
- Environment variables & profiles

## Boot & Recovery [[linux.basics.boot]]

- GRUB basics
- Rescue & emergency mode
- Single-user mode
- Basic kdump setup

## Logging [[linux.basics.logging]]

- rsyslog basics
- journald basics
- Log rotation (logrotate)

--- 
# Specialized Linux Topics
[[bash]] [[satellite]] [[ansible]]
### Security & Compliance (Regulated/Banking Focus) [[linux.security]]
### High Availability & Clustering [[linux.ha]]

### Networking (Advanced) [[linux.networking]]
## Identity & Access Management [[linux.identity]]
## Kernel Internals [[linux.niche.kernel]]

- kdump & crash utility analysis
- Kernel module loading, signing, blacklisting
- eBPF & bpftrace
- cgroups v1 vs v2 internals
- Namespaces (mnt, pid, net, uts, ipc, user)
- sysctl tuning (vm, net, kernel subsystems)
- NUMA internals & numactl
- Huge pages & THP behavior
- OOM killer tuning & scoring
- IRQ affinity & /proc/interrupts
- CPU affinity (taskset, cset)
- Kernel boot parameters (GRUB tuning)
- initramfs / dracut internals

## Process & Memory Management [[linux.niche.ps]]

- /proc & /sys deep dive
- strace / ltrace profiling
- perf & flame graphs
- SystemTap scripting
- Memory map analysis (pmap, smaps)
- Shared memory (SHM) & tmpfs internals
- Swap behavior & swappiness tuning

## Storage Internals [[linux.niche.storage]]

- VFS layer internals
- udev rules authoring
- dm-multipath internals & tuning
- LVM internals (metadata, thin pools, snapshots)
- I/O schedulers (mq-deadline, kyber, bfq)
- blktrace & blkparse
- SCSI error handling & recovery tuning
- Filesystem internals (XFS journal, ext4 extents)
- NVMe queue depth tuning

## Networking (Linux Stack Only) [[linux.niche.net]]

- Netfilter / nftables internals
- TC (traffic control) & qdiscs
- Network namespaces
- Bonding & team driver internals
- Kernel routing table (ip rule, ip route)
- Socket tuning (/proc/sys/net)
- XDP & AF_XDP
- IPVS (LVS)

## Security (Linux-Specific) [[linux.niche.sec]]

- SELinux policy writing & troubleshooting
- AppArmor profiles
- seccomp filters
- Linux capabilities internals
- Landlock
- IMA/EVM (integrity measurement)
- PAM stack deep dive
- LUKS/dm-crypt internals
- FIPS mode on RHEL
- Auditd & syscall auditing
- /etc/security/* hardening

## Systemd Deep Dive [[linux.niche.systemd]]

- Unit file authoring (all unit types)
- Systemd resource control (cgroup delegation)
- Systemd-journald internals
- Systemd-networkd & resolved
- Systemd timers
- Systemd-nspawn
- Socket activation
- Drop-ins & overrides

## Boot & Init [[linux.niche.boot]] 

- GRUB2 internals & rescue
- UEFI/Secure Boot on Linux
- dracut modules & initramfs debugging
- Early boot troubleshooting (rd.break, emergency targets)

## Filesystem & Mount [[linux.niche.fs]]

- bind mounts & mount namespaces
- Overlay filesystem
- autofs
- NFS client internals & tuning
- inotify & fanotify
- Extended attributes & ACLs

## RPM & Package Internals [[linux.niche.rpm]]

- RPM database internals
- Spec file authoring
- Creating & signing custom repos
- DNF plugin development
- Module streams (AppStream)










