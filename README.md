# ha-setup

## ESXi Setup

- Clonezilla Download: https://clonezilla.org/downloads.php (get the ISO)
- Rufus Download: https://rufus.ie
- Download ESXi + Free license: https://my.vmware.com/de/web/vmware/evalcenter?p=free-esxi7
- Fixed Failed boot (F12 on Dell -> make legacy boot) -> https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-D1BD27AB-C432-454D-9B2B-DC04E7BA9979.html

## Base HA Setup in ESXi

- Download VMDK: "Windows setup" (duno what this has to do with Windows...) - https://www.home-assistant.io/installation/windows
- HA installation in ESXi (Disk needs to be attached as IDE0, EFI Boot must be enabled - https://community.home-assistant.io/t/hass-io-on-vmware-esxi-6-7-step-by-step/151419

