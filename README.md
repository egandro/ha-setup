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


### Pairing 

You need to pair modern cell phones in order to allow tracking. The not tracked BT Mac is randomized.

- Tutorial https://kofler.info/bluetooth-konfiguration-im-terminal-mit-bluetoothctl/ (German)
- You need to "trust" the MAC in order to allow automatic reconnect on boot

### BT Audio

- unsolved - sort of sucks with HA

## Supervisor

- All add-ons are installed with start at boot, watchdog, autoupdate, show in taskbar

### File Editor

- default setup - no config file entries

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
  
- test with ssh, putty, scp, WinSCP, Visual Studio Code with the "ms-vscode-remote.remote-ssh" plugin
  
### Almond
   
- default setup - no config file entries
    
### ESPHome
  
- default setup - no config file entries
  
Link the secrets.yml file:
  
```
# ~ $ cd config
# config $ cd esphome/
# esphome $ ln -s ../secrets.yaml secrets.yaml
```

The ESPHome samples requires some settings in the secrets.yaml
  
```
# ESPHome needs the " in strings
wifi_ssid: "My_Wifi_SSID"
wifi_password: "0123456789"

esp_home_assistant_api_pw: "0123456789"
esp_home_fallback_ap_pw: "0123456789"
```
  
- Check ESPHome Template folder for templates.
- Enable the entities in Configuration / Integration and use the ```esp_home_assistant_api_pw``` to connect
  
### InfluxDB
  
- default setup - no config file entries
- open the UI
- InfluxDB Admin / Databases / Create Database "homeassistant" (duration as you wish)
- InfluxDB Admin / Users / Create User "homeassistant" (use a strong passwort, put this in secrets.yaml - key influxdb_password)
- InfluxDB Admin / Users / click user "homeassistant" / click permission / click "ALL" / click "Apply"
- Add this to configuration.yaml

```
# https://dummylabs.com/post/2019-01-13-influxdb-part1/
# https://www.home-assistant.io/integrations/influxdb/
# https://docs.influxdata.com/influxdb/cloud/security/tokens/create-token/
influxdb:
  host: a0d7b954-influxdb
  port: 8086
  database: homeassistant
  username: homeassistant
  password: !secret influxdb_password
  max_retries: 3
  default_measurement: state
```

- Restart Homeassistant and check if you get values in the Database.
- open the UI
- click "Explore"
- click "homeassistant.autogen"
- you should see some of the data that InfluxDB already collected


### Grafana
  
- Make sure InfluxDB is configured and created some data
- default setup - no config file entries
- open the UI
- Configuration / Add data source / InfluxDB
- HTTP / URL: http://a0d7b954-influxdb:8086
- InfluxDB Details / Database: homeassistant
- InfluxDB Details / User: homeassistant
- InfluxDB Details / Password: (secrets.yaml - influxdb_password key)
- Click Save & Test
  
Now you can configure your dashboards.
  
### WireGuard Client
  
- This is no standard add on.
- Supervisor / Add-on Store - click on top right burger menu / enter "https://github.com/bigmoby/hassio-repository-addon"
- Install "Home Assistant Bigmoby Add-ons"
  
#### Configure Debian as WireGuard Server

- Reference: https://www.cyberciti.biz/faq/debian-10-set-up-wireguard-vpn-server/
  
```
$ sudo apt update
$ sudo apt upgrade
$ sudo sh -c "echo 'deb http://deb.debian.org/debian buster-backports main contrib non-free' > /etc/apt/sources.list.d/buster-backports.list"
$ cat /etc/apt/sources.list.d/buster-backports.list
$ sudo apt update
$ apt search wireguard
$ sudo apt install wireguard
$ sudo -i
# cd /etc/wireguard/
# umask 077; wg genkey | tee privatekey | wg pubkey > publickey
# ls -l privatekey publickey
$ cat privatekey
## Please note down the private key of the server ##
$ cat publickey
## Please note down the public key of the server ##
# exit
```
  
Unfortunately there is no nice way to create keys on client using the plugin. So we use "wg" of the server.

```
$ mkdir -p clientkey
$ cd clientkey
$ wg genkey | tee privatekey | wg pubkey > publickey
$ ls -l privatekey publickey
$ cat privatekey
## Please note down the private key of the client ##
$ cat publickey
## Please note down the public key of the client ##
$ cd .. && rm -rf clientkey
```
  
Create a wg0.conf file on the server

```
$ sudo vim /etc/wireguard/wg0.conf
[Interface]
Address = 192.168.120.1/24
ListenPort = 51820
PrivateKey = <privatekey_of_the_server>

[Peer]
PublicKey = <publickey_of_the_client>
AllowedIPs = 192.168.120.0/24
```

#### Configure HomeAssistant as WireGuard Server  
  
```
interface:
  private_key: <privatekey_of_the_client>
  address: 192.168.120.2/24
  dns:
    - 8.8.8.8
    - 8.8.4.4
  post_up: iptables -t nat -A POSTROUTING -o wg0 -j MASQUERADE
  post_down: iptables -t nat -D POSTROUTING -o wg0 -j MASQUERADE
peer:
  public_key: <publickey_of_the_server>
  endpoint: IP_OF_YOUR_SERVER:51820
  allowed_ips:
    - 192.168.120.0/24
  persistent_keep_alive: '20'
```

- Please put the "WireGuard client status API" Port to 51880 - for some crazy reason this is using 80 as default.
- you can now start the WireGuard client in HA
  
  
#### Enable / Start WireGuard on the Debian Server
  
```
$ sudo ufw allow 51820/udp # if ufw is enable
$ sudo systemctl enable wg-quick@wg0
$ sudo systemctl start wg-quick@wg0
$ sudo systemctl status wg-quick@wg0
```

#### Enable Nginx proxy on the Debian Server for LetsEncrypt Certificates
  
- IP Adress of the WireGuard is public
- The IP adress gets a CNAME e.g. my-ha.example.com
- Install Docker
- Ensure port 80 / 443 is currently not used

```
$ sudo mkdir -p /var/opt/nginx-proxy-letsencrypt  
$ sudo vi /var/opt/nginx-proxy-letsencrypt/start.sh
#!/bin/bash

mkdir -p /var/opt/nginx-proxy-letsencrypt

# new image https://github.com/nginx-proxy/nginx-proxy
docker run \
  --detach \
  --restart always \
  --publish 80:80 \
  --publish 443:443 \
  --name nginx-proxy \
  --volume /var/run/docker.sock:/tmp/docker.sock:ro \
  --volume /var/opt/nginx-proxy-letsencrypt/nginx-certs:/etc/nginx/certs \
  --volume /var/opt/nginx-proxy-letsencrypt/nginx-vhost:/etc/nginx/vhost.d \
  --volume /var/opt/nginx-proxy-letsencrypt/nginx-html:/usr/share/nginx/html \
  jwilder/nginx-proxy

# new image https://github.com/nginx-proxy/acme-companion
docker run \
  --detach \
  --restart always \
  --name nginx-proxy-letsencrypt \
  --volumes-from nginx-proxy \
  --volume /var/run/docker.sock:/var/run/docker.sock:ro \
  --volume /etc/acme.sh \
  --env "DEFAULT_EMAIL=mail@example.com" \
  jrcs/letsencrypt-nginx-proxy-companion
```

Start the nginx-proxy-companion  
```
$ sudo chmod /var/opt/nginx-proxy-letsencrypt/start.sh
$ sudo /var/opt/nginx-proxy-letsencrypt/start.sh  
```
  
Create a 2nd proxy for forwarding to the WG VPN. This wil automaticually query the SSL certificat.
  
```
$ sudo mkdir -p /var/opt/nginx-proxy-letsencrypt/my-ha.example.com
$ sudo vi /var/opt/nginx-proxy-letsencrypt/my-ha.example.com/vpn.conf
upstream vpn {
  server        192.168.120.2:8123;
}

server {
  listen        80;
  server_name   my-ha.example.com;

  location / {
    proxy_pass  http://vpn/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```

Create the start script
  
```
$ sudo vi /var/opt/nginx-proxy-letsencrypt/my-ha.example.com.sh
#!/bin/bash
docker run \
  --detach \
  --restart always \
  --name my-ha.example.com \
  --env VIRTUAL_HOST=my-ha.example.com \
  --env LETSENCRYPT_HOST=my-ha.example.com \
  --env LETSENCRYPT_EMAIL="youremail@my-ha.example.com" \
  --volume /var/opt/nginx-proxy-letsencrypt/my-ha.example.come/vpn.conf:/etc/nginx/conf.d/default.conf \
  nginx:latest
```
  
Start the nginx proxy  
```
$ sudo chmod /var/opt/nginx-proxy-letsencrypt/my-ha.example.com.sh
$ sudo /var/opt/nginx-proxy-letsencrypt/my-ha.example.com.sh  
```

#### Configure HA to allow to be connected via the WireGuard tunnel on my-ha.example.com.sh
  
- Add this to configuration.yaml

```
# wireguard proxy
# https://www.home-assistant.io/integrations/http#use_x_forwarded_for
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - !secret trusted_proxy_ip
```
  
- put trusted_proxy_ip: in secret.yaml with 192.168.120.1   
- restart
- test if https://my-ha.example.com works


#### Configure HA internal and external URLs
  
- Configuration / General
- Scroll Down
- External URL: https://my-ha.example.com/
- Internal URL: http://homeassistant.local:8123/ 
  
  
## Install HACS

- This is a super cool community repository. It contains add-ons, plugins, drivers, ...  
- https://hacs.xyz/docs/installation/installation
- Please install the "SSH & Web Terminal" in the Supervisor section of this document
- SSH into HA  
```
$ wget -q -O - https://install.hacs.xyz | bash -
```
- Restart HA
- Clear Browser Cache
- Enable the HACS Integration: https://hacs.xyz/docs/configuration/basic
- Configuration / Integrations / "+" Button / Search for HACS
- Wait ... confirm the 4 checkboxes of the dialog ...
- Authenticate the Device to Github / your Github account
- Click "HACS" on the left bar 
- Starting "HACS" the first time will take a while

  
## Additional Setup Steps
  
### SSH Keys (required to run scripts from HA -> other systems)
  
Put your ssh keys (private/pub) to ```/root/config/ssh_keys```
  
```
$ chmod 600 /root/config/ssh_keys/*
```

### Force the lovelace Dashboard to be the default one

- Add this to configuration.yaml
  
```
# https://www.home-assistant.io/lovelace/dashboards-and-views
lovelace:
  mode: storage
```

### Config Check
  
- Adds a better service to check the configuraton e.g. in services
- can be installed via HACS
- https://github.com/custom-components/config_check


### Auto Backup
  
- Cron like backup of the configuration
- can be installed via HACS
- https://github.com/jcwillox/hass-auto-backup
  
  
## Setup Entities  
  
### Enable BT Tracking (cell phone)
  
- make sure you paired your device and "trust" is enabled on HA in CLI
- Add this to configuration.yaml

```
# https://community.home-assistant.io/t/ble-tracking-with-raspberry-pi-4/155593/9
device_tracker:
#  - platform: bluetooth_le_tracker
#    interval_seconds: 10
#    consider_home: 10
#    new_device_defaults:
#      track_new_devices: true
  - platform: bluetooth_tracker
    interval_seconds: 10
    consider_home: 30
    # enable this after your devices are found
    #new_device_defaults:
    #  track_new_devices: false
```

### Ping hosts

- Add this to configuration.yaml
- please note: pinging a cell phone might not work because of powersafe mode    
  
```
binary_sensor: 
  - platform: ping #https://www.home-assistant.io/integrations/ping/
    host: 192.168.100.114
    name: "coffemaker_connected"
    count: 2
    scan_interval: 30
```
  
### FireTV / Android TV Boxes
  
- Add this to configuration.yaml
  
```
media_player:
  # Use the Python ADB implementation
  - platform: androidtv
    name: Fire TV
    host: 192.168.100.42
    device_class: firetv
    #adb_server_ip: 127.0.0.1
    #adb_server_port: 5037
    get_sources: false
```

- Turn on you TV / Screen
- Turn on your FireTV / Android TV Box
- Enable remote debugging (check Fire TV docs)
- Restart Homeassistant
- If this is the first time an Android Debug dialog will pop up to trust the hostkey of Home assistant.

### Text to speech local language support
  
- Add this to configuration.yaml
  
```
# Text to speech
tts:
  - platform: google_translate
    language: "de"
```  
  
### Show local IP
  
- Configuration / Integrations / +
- Local  IP Address
  
### Tapo: Cameras Control
  
- Diver for Tapo Cam (C200, C210): https://github.com/JurajNyiri/HomeAssistant-Tapo-Control
- Install HACS
- HACS -> Explore & Add Repositories
- Search for "Tapo: Camera Control"
- Restart Server
- Clear Browser Cache
- Configuration / Integrations / + / Tapo: Cameras Control (use main branch)
- You will need the email/passwort of the app
- sample configuration.yml entry to enable/disable camera
  
```
switch:  
   # https://community.home-assistant.io/t/make-scripts-as-toggle-switch/158688/11
  - platform: template
    switches:
      c210_enabled_mode:
        friendly_name: C210 enabled 
        value_template: "{{ is_state_attr('camera.tapo_camera_c956_hd', 'privacy_mode', 'off') }}"
        turn_on:
          service: tapo_control.set_privacy_mode
          target:
            entity_id: camera.tapo_camera_c956_hd
          data:
            privacy_mode: 'off'
        turn_off:
          service: tapo_control.set_privacy_mode
          target:
            entity_id: camera.tapo_camera_c956_hd
          data:
            privacy_mode: 'on'
 ```
  
### Tapo: P100 Smart Plug
  
- Diver for Tapo P100 Smart Plug: https://github.com/fishbigger/HomeAssistant-Tapo-P100-Control
- Setup the Plug with the app
- Block all traffic to the internet on your router
- SSH into HA
  
```
$ cd config
$ cd custom_components
$ wget -O tapo_p100_control.zip https://github.com/fishbigger/HomeAssistant-Tapo-P100-Control/archive/refs/heads/main.zip
$ unzip tapo_p100_control.zip
$ mv HomeAssistant-Tapo-P100-Control-main/tapo_p100_control .
$ rm tapo_p100_control.zip
$ rm -rf HomeAssistant-Tapo-P100-Control-main
```

- Restart Server
- You will need the email/passwort of the app
- put it in secret.yaml 
  - tp_tapo_email: "might be the same as for the cam cloud password"
  - tp_tapo_password: "might be the same as for the cam cloud password"
- sample configuration for configuration.yaml
  
```
switch:
  - platform: tapo_p100_control
    ip_address: 192.168.85.151
    email: !secret tp_tapo_email
    password: !secret tp_tapo_password
```
  
### Broadlink (IR Sender / IP Remote control)
  
- Install App and configure
- Configuration / Integrations / Enable the device (it should be auto detected)
- main configuration is a lot of RTFM
- Here posible sample of /config/scripts.yaml
  
```
tv_power:
  sequence:
  - service: remote.send_command
    data:
      command: b64:<however you get this>
      device: tv_power
    target:
      entity_id: remote.broadlink_remote
  mode: single
  alias: TV Power
  icon: hass:power-standby
```

### Turn On/Off PC
  
- Enable / configure WOL (Bios, Operating System, ...)
- Configure SSH login with SSH Keys
  - Windows Version with Choco ```choco install mls-software-openssh```
  - https://community.chocolatey.org/packages/mls-software-openssh
- Put the public key off /root/config/ssh_keys/ as authorized_key on the target machine
- Login to HA and test if the keys are working ```ssh -i /config/ssh_keys/id_rsa -p 22 -o StrictHostKeyChecking=no user@192.168.134.24```
- sample config for configuration.yaml
  
```
switch:
  - platform: wake_on_lan
    name: PC
    mac: ab:cd:ef:11:22:33
    host: "192.168.51.1"
    broadcast_address: "192.168.51.255"
    turn_off:
     service: shell_command.turn_off_pc

shell_command:
  # Windows version
  turn_off_pc: "ssh -i /config/ssh_keys/id_rsa -p 22 -o StrictHostKeyChecking=no user@192.168.134.24 rundll32.exe powrprof.dll,SetSuspendState 0,1,0"
  ### # Linux version #https://www.cyberciti.biz/faq/linux-command-to-suspend-hibernate-laptop-netbook-pc/
  ### turn_off_pc: "ssh -i /config/ssh_keys/id_rsa -p 22 -o StrictHostKeyChecking=no user@192.168.134.24 sudo pm-hibernate"
```
  
### Generic Shell based switch
  
- This might be stupid, as the state is saved "on/off"
- You might want to go with a script or a shell command
- sample config for configuration.yaml
  
```
switch:
  - platform: command_line
    switches:
      my_cool_switch:
         friendly_name: The coolest switch
         command_on: "ssh -i /config/ssh_keys/id_rsa -p 22 -o StrictHostKeyChecking=no user@192.168.134.24 whatever"
         command_off: "ssh -i /config/ssh_keys/id_rsa -p 22 -o StrictHostKeyChecking=no user@192.168.134.24 whatever"
```
  
### ESXi statistics plugin

- needs HACS
- https://github.com/wxt9861/esxi_stats
- needs hacks :( https://github.com/wxt9861/esxi_stats/issues/57

## Lovelace customizing
  
### lovelace_gen

- Adds a super simple handling of lovelace dashboards
- can be installed via HACS
- https://github.com/thomasloven/hass-lovelace_gen
- After installing via HACS - restart Home Assistant - after that
- Add this to configuration.yaml
  
```
lovelace_gen:
```  
  
- Restart Home Assistant again
- Configuration / Lovelace Dashboards / Add
- create a new Dashboard, click on the Name (not Edit) "Make this default on this device"
- With edit you can customize / add tabs
  
## Tools
  
- official icons: https://materialdesignicons.com/icon/
