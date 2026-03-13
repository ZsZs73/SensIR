# Setup
Connect to the failback AP started by tasmota FW: tasmota-AABBCC-DDEE. 
Wifimaganer is waiting for 3 minutes for connection. 
Navigate to 192.168.4.1 (default ip address for tasmota failback AP).
Select your IoT Wifi AP to connect to and enter password.
The device will reboot and connect to yout IoT WiFi network. Check your DHCP leases log on your WiFi AP/Router in order to get the assigned ip address.

## Setup the device
Navigate to the ip address in a new browser window. The main page of the tasmota should show up 'ESP32C3', Tasmota
Navigate to Configuration / Module / Module type drop-down list
- Select ESYQ3A
- Click 'Save'
The device will reboot
Navigate to Configuration / Other
- Device Name: 
TasmotaESYQ3A
- Friednly Name: 
TasmotaESYQ3A
- Click 'Save'

GPIO08 is already configured as LedLink_i via ESYQ3A template. The firmware turns the blue LED:
- on during boot
- blink slow (on: 1s, off:1s) until the Wifi and MQTT (if configured) connections are established
- off once the WiFi AND MQTT connections are ok
info: https://tasmota.github.io/docs/Lights/#status-leds

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
smlj=0 ;Disable MQTT publish on boot to avoid sending zero values
print Disabling MQTT on boot
>R
smlj=0 ;Disable MQTT publish on reboot to avoid sending zero values
print Disabling MQTT on restart
>S
if ((sml[1]==0) and smlj==0) { ; Enable MQTT only when non-zero SML data received
  print Waiting for non-zero SML data
  break
} else {
  print Enabling MQTT
  smlj=1 ; Enable MQTT publish
  print Sending Autodiscovery message
  =>setoption19 0 ; Send autodiscovery message via MQTT
}
>M 1
+1,10,s,0,9600,ESYQ3A
1,77070100010800ff@1000,Consumed_energy,kWh,Consumed,5
1,77070100020800ff@1000,Exported_energy,kWh,Exported,5
1,=h-- 
1,77070100100700ff@1,Power_sum,W,Watt_sum,2
1,77070100240700ff@1,Power_L1,W,Watt_L1,2
1,77070100380700ff@1,Power_L2,W,Watt_L2,2
1,770701004c0700ff@1,Power_L3,W,Watt_L3,2
1,=h-- 
1,77070100200700ff@1,Voltage_L1,V,Volt_L1,1
1,77070100340700ff@1,Voltage_L2,V,Volt_L2,1
1,77070100480700ff@1,Voltage_L3,V,Volt_L3,1
#

```
then click Save

MQTT messages sent out with ths method:
```
14:09:59.306 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:09:59","SML":{"Zaehler_Nummer":"1ESY1162XXXXXX","Consumed_energy":3094.6729817,"Exported_energy":8571.9683706,"Power_sum":114.87,"Power_L1":13.05,"Power_L2":187.43,"Power_L3":-59.50,"Voltage_L1":231.0,"Voltage_L2":230.6,"Voltage_L3":232.6}}
14:10:09.258 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:10:09","SML":{"Zaehler_Nummer":"1ESY1162XXXXXX","Consumed_energy":3094.6733116,"Exported_energy":8571.9683706,"Power_sum":118.07,"Power_L1":12.82,"Power_L2":188.31,"Power_L3":-57.41,"Voltage_L1":231.0,"Voltage_L2":230.3,"Voltage_L3":232.1}}
```
### +16 method
This method will send out every value as a single mqtt message upon measurement received from the meter. This way you can get data with 1 sec resolution.
Navigate to Tools/Edit Script check 'Script enable' and paste the following (the line with # is important!):
```
>D
>B
=>sensor53 r
smlj=0 ;Disable MQTT publish on boot to avoid sending zero values
print Disabling MQTT on boot
>R
smlj=0 ;Disable MQTT publish on reboot to avoid sending zero values
print Disabling MQTT on restart
>S
if ((sml[1]==0) and smlj==0) { ; Enable MQTT only when non-zero SML data received
  print Waiting for non-zero SML data
  break
} else {
  print Enabling MQTT
  smlj=1 ; Enable MQTT publish
  print Sending Autodiscovery message
  =>setoption19 0 ; Send autodiscovery message via MQTT
}
>M 1
+1,10,s,0,9600,ESYQ3A
1,77070100010800ff@1000,Consumed_energy,kWh,Consumed,21
1,77070100020800ff@1000,Exported_energy,kWh,Exported,21
1,=h-- 
1,77070100100700ff@1,Power_sum,W,Watt_sum,18
1,77070100240700ff@1,Power_L1,W,Watt_L1,18
1,77070100380700ff@1,Power_L1,W,Watt_L2,18
1,770701004c0700ff@1,Power_L1,W,Watt_L3,18
1,=h-- 
1,77070100200700ff@1,Voltage_L1,V,Volt_L1,18
1,77070100340700ff@1,Voltage_L2,V,Volt_L2,18
1,77070100480700ff@1,Voltage_L3,V,Volt_L3,18
#

```
then click Save

MQTT messages sent out with ths method:
```

14:12:02.524 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Consumed_energy":3094.6765688}}
14:12:02.587 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Exported_energy":8571.9683706}}
14:12:02.595 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Power_sum":90.74}}
14:12:02.605 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Power_L1":23.41}}
14:12:02.629 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Power_L1":183.65}}
14:12:02.654 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Power_L1":-69.49}}
14:12:02.775 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Voltage_L1":230.8}}
14:12:02.793 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Voltage_L2":230.1}}
14:12:02.812 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:02","SML":{"Voltage_L3":233.0}}
14:12:03.524 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Consumed_energy":3094.6765947}}
14:12:03.552 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Exported_energy":8571.9683706}}
14:12:03.580 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Power_sum":93.16}}
14:12:03.605 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Power_L1":22.95}}
14:12:03.666 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Power_L1":184.99}}
14:12:03.674 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Power_L1":-68.88}}
14:12:03.775 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Voltage_L1":230.7}}
14:12:03.793 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Voltage_L2":230.0}}
14:12:03.812 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:03","SML":{"Voltage_L3":232.9}}
14:12:04.523 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Consumed_energy":3094.6766217}}
14:12:04.551 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Exported_energy":8571.9683706}}
14:12:04.580 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Power_sum":97.28}}
14:12:04.605 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Power_L1":22.79}}
14:12:04.630 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Power_L1":186.97}}
14:12:04.655 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Power_L1":-66.89}}
14:12:04.775 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Voltage_L1":230.8}}
14:12:04.793 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Voltage_L2":230.2}}
14:12:04.812 MQT: tele/tasmota_DC0386/SENSOR = {"Time":"2025-09-22T14:12:04","SML":{"Voltage_L3":232.8}}
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


