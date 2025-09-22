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

## Send data to MQTT
Navigate to Configuration / MQTT and enter the followoing data:
- Host: your MQTT server (IP or FQDN)
- Port: change if it is not the default 1883
- User: enter the mqtt username
- Password: enter the mqtt user password
### Teleperiod method
The script will read the SML data from pin RX decode the data and send all readings to the MQTT server every 10 seconds. Note that the lowest possible value is 10 secs.

Navigate to Tools/Edit Script check 'Script enable' and paste the following (the line with # is important!):
```
>D
>B
=>sensor53 r
=>teleperiod 10
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

MQTT messages sent out with ths method:
```
14:09:59.306 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:09:59","SML":{"Zaehler_Nummer":"1ESY1162XXXXXX","Verbrauch_Summe":3094.6729817,"Einspeisung_Summe":8571.9683706,"Watt_Summe":114.87,"Watt_L1":13.05,"Watt_L2":187.43,"Watt_L3":-59.50,"Volt_L1":231.0,"Volt_L2":230.6,"Volt_L3":232.6}}
14:10:09.258 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:10:09","SML":{"Zaehler_Nummer":"1ESY1162XXXXXX","Verbrauch_Summe":3094.6733116,"Einspeisung_Summe":8571.9683706,"Watt_Summe":118.07,"Watt_L1":12.82,"Watt_L2":188.31,"Watt_L3":-57.41,"Volt_L1":231.0,"Volt_L2":230.3,"Volt_L3":232.1}}
```
### +16 method
This method will send out every value as a single mqtt message upon measurement received from the meter. This way you can get data with 1 sec resolution.
Navigate to Tools/Edit Script check 'Script enable' and paste the following (the line with # is important!):
```
>D
>B
=>sensor53 r
>M 1
+1,3,s,0,9600,SML
1,=so2,4 ;invert RX
1,77070100000000ff@#x,Zaehler_Nummer,,Zaehler_Nummer,0
1,77070100010800ff@1000,Verbrauch_Summe,kWh,Verbrauch_Summe,23
1,77070100020800ff@1000,Einspeisung_Summe,kWh,Einspeisung_Summe,23
1,=h-- 
1,77070100100700ff@1,Wirkleistung_Summe,W,Watt_Summe,18
1,77070100240700ff@1,Wirkleistung_L1,W,Watt_L1,18
1,77070100380700ff@1,Wirkleistung_L2,W,Watt_L2,18
1,770701004c0700ff@1,Wirkleistung_L3,W,Watt_L3,18
1,=h-- 
1,77070100200700ff@1,Spannung_L1,V,Volt_L1,17
1,77070100340700ff@1,Spannung_L2,V,Volt_L2,17
1,77070100480700ff@1,Spannung_L3,V,Volt_L3,17
#
```
then click Save

MQTT messages sent out with ths method:
```

14:12:02.524 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Verbrauch_Summe":3094.6765688}}
14:12:02.587 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Einspeisung_Summe":8571.9683706}}
14:12:02.595 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Watt_Summe":90.74}}
14:12:02.605 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Watt_L1":23.41}}
14:12:02.629 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Watt_L2":183.65}}
14:12:02.654 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Watt_L3":-69.49}}
14:12:02.775 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Volt_L1":230.8}}
14:12:02.793 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Volt_L2":230.1}}
14:12:02.812 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Volt_L3":233.0}}
14:12:03.524 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Verbrauch_Summe":3094.6765947}}
14:12:03.552 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Einspeisung_Summe":8571.9683706}}
14:12:03.580 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Watt_Summe":93.16}}
14:12:03.605 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Watt_L1":22.95}}
14:12:03.666 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Watt_L2":184.99}}
14:12:03.674 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Watt_L3":-68.88}}
14:12:03.775 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Volt_L1":230.7}}
14:12:03.793 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Volt_L2":230.0}}
14:12:03.812 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Volt_L3":232.9}}
14:12:04.523 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Verbrauch_Summe":3094.6766217}}
14:12:04.551 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Einspeisung_Summe":8571.9683706}}
14:12:04.580 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Watt_Summe":97.28}}
14:12:04.605 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Watt_L1":22.79}}
14:12:04.630 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Watt_L2":186.97}}
14:12:04.655 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Watt_L3":-66.89}}
14:12:04.775 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Volt_L1":230.8}}
14:12:04.793 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Volt_L2":230.2}}
14:12:04.812 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Volt_L3":232.8}}
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


