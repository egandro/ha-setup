# Tasmota OTA on a Gosund SP112



## Flash

- Tutorial https://www.youtube.com/watch?v=hfYFB1UENTQ (German)
- Tutorial https://www.tasmota.info/sp112-flash/ (German)
- Firmware: http://ota.tasmota.com/tasmota/release/
- Tasmotizer: https://github.com/tasmota/tasmotizer/releases

## Flash (OTA - won't work on new devices)


- Tutorial: https://www.youtube.com/watch?v=tx6T4C9w-lw (German)

Code: (on RaspianOS - you need a cell phone as WIFI Bridge)

```
sudo apt-get update
sudo apt-get upgrade
 
sudo apt-get install git
 
git clone https://github.com/ct-Open-Source/tuya-convert
 
cd tuya-convert
 
sudo ./install_prereq.sh
 
sudo ./start_flash.sh
```

## Setup

- Tutorial: https://www.youtube.com/watch?v=qIY7EWpsICM (German)

```
Tamplate SP112: 
{"NAME":"Gosund 112v3.4","GPIO":[56,0,57,0,132,134,0,0,131,30,21,0,0],"FLAG":4,"BASE":18}
{"NAME":"SP112new","GPIO":[57,0,56,0,132,134,0,0,131,30,21,0,0],"FLAG":4,"BASE":18}
{"NAME":"SHP5","GPIO":[57,145,56,146,0,22,0,0,0,0,21,0,17],"FLAG":0,"BASE":18}
```

```
VoltageSet  (V)
CurrentSet  (mA e.g.: 0,65A= 650mA)
PowerSet  (W)
```
