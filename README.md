# ha-setup

## ESXi Setup

- Clonezilla Download: https://clonezilla.org/downloads.php (get the ISO)
- Rufus Download: https://rufus.ie
- Download ESXi + Free license: https://my.vmware.com/de/web/vmware/evalcenter?p=free-esxi7
- Fixed Failed boot (F12 on Dell -> make legacy boot) -> https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-D1BD27AB-C432-454D-9B2B-DC04E7BA9979.html

## Base HA Setup in ESXi

- Download VMDK: "Windows setup" (duno what this has to do with Windows...) - https://www.home-assistant.io/installation/windows
- Add a USB Controller 2.0
- HA installation in ESXi (Disk needs to be attached as IDE0, EFI Boot must be enabled - https://community.home-assistant.io/t/hass-io-on-vmware-esxi-6-7-step-by-step/151419


## Setup Bluetooth in ESXi

- I inserted a BT Dongle (3-5€ Amazon/Ebay BT 3,4,5...) to one of the ESXi BT Ports
- In the Edit Settings tab add the detectd USB Device to the VM. https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.vm_admin.doc/GUID-507F65CD-E855-4527-B076-567F27C98A29.html


### Paring

You need to pair modern cell phones in order to allow tracking. The not tracked BT Mac is randomized.

- Tutorial https://kofler.info/bluetooth-konfiguration-im-terminal-mit-bluetoothctl/ (German)
- You need to "trust" the MAC in order to allow automatic reconnect on boot

### BT Audio

- unsolved - sort of sucks with HA


## Supervisor

### File Editor

- default setup - no config

### SSH & Web Terminal

- <User> / Advanced Mode on
- Install the "SSH & Web Terminal" (from Home Assistant Community Add-ons)
- We need the ```compatibility_mode: true``` in order to make the MS SSH FS plugin to working: ms-vscode-remote.remote-ssh
  
```
ssh:
  username: root
  password: ''
  authorized_keys: ['add your key here']
  sftp: true
  compatibility_mode: true
  allow_agent_forwarding: false
  allow_remote_port_forwarding: false
  allow_tcp_forwarding: false
zsh: true
share_sessions: false
packages: []
init_commands: []
```
  
### Almond
   
- default setup - no config
    
### ESPHome
  
