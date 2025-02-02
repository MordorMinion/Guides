Cataloging how I was able to reach < C2 states on Proxmox with consumer hardware

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
