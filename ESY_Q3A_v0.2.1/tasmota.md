# Setup
Connect to the failback AP started by tasmota FW: tasmota-AABBCC-DDEE. 
Wifimaganer is waiting for 3 minutes for connection. 
Navigate to 192.168.4.1 (default ip address for tasmota failback AP).
Select your IoT Wifi AP to connect to and enter password.
The device will reboot and connect to yout IoT WiFi network. Check your DHCP leases log on your WiFi AP/Router in order to get the assigned ip address.

## Setup the device
Navigate to the ip address in a new browser window. The main page of the tasmota should show up 'Sonoff Basic', Tasmota
Navigate to Configuration / Module
- Select Generic (0)
- Select D4 GPIO02: 'LedLink_i'
Configuring GPIO02 as LedLink_i turns the blue LED on the ESP12 module:
- on during boot
- blink until the Wifi and MQTT (if configured) connections are established
- off once the connections are ok

info: https://tasmota.github.io/docs/Lights/#status-leds
Click Save, the device will reboot

## Script
This script wil read the SML data from pin RX and decode the data

Navigate to Tools/Edit Script check 'Script enable' and paste the following (the line with # is important!):
```
>D
>B
=>sensor53 r
>M 1
+1,3,s,0,9600,SML
1,=so2,4 ;invert RX
1,77070100000000ff@#x,Zaehler_Nummer,,Zaehler_Nummer,0
1,77070100010800ff@1000,Verbrauch_Summe,kWh,Verbrauch_Summe,7
1,77070100020800ff@1000,Einspeisung_Summe,kWh,Einspeisung_Summe,7
1,=h-- 
1,77070100100700ff@1,Wirkleistung_Summe,W,Watt_Summe,2
1,77070100240700ff@1,Wirkleistung_L1,W,Watt_L1,2
1,77070100380700ff@1,Wirkleistung_L2,W,Watt_L2,2
1,770701004c0700ff@1,Wirkleistung_L3,W,Watt_L3,2
1,=h-- 
1,77070100200700ff@1,Spannung_L1,V,Volt_L1,1
1,77070100340700ff@1,Spannung_L2,V,Volt_L2,1
1,77070100480700ff@1,Spannung_L3,V,Volt_L3,1
#
```
then click Save

# Set publishing interval
via Console
```
TelePeriod 10
```

via script
```
tper=10
```

# Enable Serial debug
Navigate to Tools/Console and enter the following command:
```
sensor53 d1
```
Diesble Serial debug
```
sensor53 d0
```


