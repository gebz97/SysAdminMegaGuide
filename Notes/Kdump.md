### Disable auto reset
```bash
vim /etc/kdump.conf
```
```conf
auto_reset_crashkernel=no
```
### Update grubby remove kdump
```bash
grubby --update-kernel=/boot/vmlinuz-$(uname -r) --remove-args="crashkernel"
```
```bash
grubby --update-kernel=ALL --remove-args="crashkernel"
```
