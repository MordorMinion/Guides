Notes on ASPM adventures 
I have currently only been able to reach C3.

System
| Component   | Name                    |  Out of the box ASPM  | Modified to work |
| ----------- | ----------------------- | --------------------- | ---------------- |
| Motherboard | ASUS Prime H670 Plus D4 | N/A                   |                  |
| CPU         | Intel i3-13100T         | N/A                   |                  |
| RAM         | 4x16GB DDR4 3200MHz     | N/A                   |                  |
| NVME        | (1) Samsung 980 512GB   | Yes                   |                  |
| SSD         | (1)  PM863a 1.92TB SATA | Unknown               |                  |
| HBA         | Dell HBA330             | No                    |                  |
| HDD         | (4) ST18000NM4J         | N/A                   | OpenSeaChest     |
| Eth         | Realtek RTL8125         | No                    |                  |

Kernel Parameters:
CONFIG_PCIEASPM=y
pcie_aspm=force
pcie=noaer
pcie=nommconf
pcie=nomsi
pcie_aspm.policy=powersave
pcie_aspm.enable=1

pcie=nomsi: Disables Message Signaled Interrupts (MSI) and forces the use of legacy IRQs. While not directly related to ASPM, MSI can sometimes interact with power management features.
pcie=noaer: Disables Advanced Error Reporting (AER) for PCI Express. AER helps in detecting and reporting hardware errors, and disabling it might be useful for troubleshooting certain power-related issues, but it's generally not recommended for normal operation.
pcie=nommconf: Disables PCIe enhanced configuration mechanism (MMCONFIG). This is a legacy option and generally not needed on modern systems. It might be relevant in very specific compatibility scenarios but is unlikely to directly affect ASPM in most cases.

Commands 
echo -n powersave > /sys/module/pcie_aspm/parameters/policy
powertop --auto-tune
echo "powersave" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
lspci -vv | awk '/ASPM/{print $0}' RS= | grep --color -P '(^[a-z0-9:.]+|ASPM )'

Guides:
- https://blog.kumo.dev/2022/12/07/realtek_drivers_proxmox.html & https://forums.unraid.net/topic/141349-plugin-realtek-r8125-r8126-r8168-and-r815267-drivers/page/9/#comment-1379870
- https://github.com/notthebee/AutoASPM
- https://www.reddit.com/r/debian/comments/8c6ytj/active_state_power_management_aspm/
- https://www.reddit.com/r/homelab/comments/1ck60qp/debugging_why_aspm_is_disabled/
- http://ftp.dei.uc.pt/pub/linux/kernel/people/mcgrof/aspm/enable-aspm

/etc/default/grub:
BOOT_IMAGE=/boot/vmlinuz-6.8.12-8-pve root=/dev/mapper/pve-root ro quiet intel_iommu=on iommu=pt pci=noaer pcie_acs_override=downstream,multifunction initcall_blacklist=sysfb_init video=simplefb:off video=vesafb:off video=efifb:off video=vesa:off disable_vga=1 vfio_iommu_type1.allow_unsafe_interrupts=1 kvm.ignore_msrs=1 modprobe.blacklist=radeon,nouveau,nvidia,nvidiafb,nvidia-gpu pcie_aspm=force


Resources:
- https://github.com/lanceshelton/irqstat (How to tell what is waking the system)
- https://forums.servethehome.com/index.php?threads/is-aspm-working.45359/
```
# even though disabled in the BIOS, powertop thinks that the onboard ethernet adapters still have wakeup functions enables. so we switch this off.
echo disabled > /sys/class/net/eno1/device/power/wakeup
echo disabled > /sys/class/net/eno2/device/power/wakeup

# cpu freq scaling governor using intel_pstate.
echo powersave | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor > /dev/null

# enery <-> performance bias.
echo balance_power | tee /sys/devices/system/cpu/cpu*/cpufreq/energy_performance_preference > /dev/null

# aspm power saving.
echo powersupersave > /sys/module/pcie_aspm/parameters/policy

# powertop thinks some NMI watchdog is enabled. this switches the watchdog off.
echo 0 > /proc/sys/kernel/nmi_watchdog

# optimisation from powertop.
echo 1500 > /proc/sys/vm/dirty_writeback_centisecs

# prefer lower power consumption over more performance.
echo 15 | tee /sys/devices/system/cpu/cpu*/power/energy_perf_bias > /dev/null
```


NVME interrupts were HUGE, attempting to follow Andrew H's answer here: 
- https://serverfault.com/questions/1052448/how-can-i-override-irq-affinity-for-nvme-devices/1063201#1063201
