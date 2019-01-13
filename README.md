Jim's EdgeX Demo
================
Dec 28, 2018
This is my EdgeX Demo using my SNMP and MQTT Device services.

- https://github.com/jpwhitemn/device-snmp-patlite
- https://github.com/jpwhitemn/device-mqtt-go

And then the regular EdgeX services (per the docker compose file in this repo)

Build
-----
Build the Docker container for the patlite & MQTT device service, and the Clojure UI (from IoTech)

(code from jpwhitemn github - fork of device-snmp-patlite-go in holding with extra config and profiles)
- docker build -t jims/docker-device-snmp-patlite:0.9.0 .
(code from jpwhitemn github - fork of device-mqtt-go in EdgeX with extra config and profiles)
- docker build -t jims/docker-device-mqtt:0.9.0 .
(code from EdgeX - no changes)
- docker build -t jims/docker-edgex-ui-clojure:0.9.0 .
(code from jpwhitemn github - for of device-modbus-go in EdgeX with extra config and profiles)
- docker build -t jims/docker-device-modbus:0.9.0 .


Run
---
Then docker compose up the services.  
Note that rules engine starts slowly and may have to be brought up after all other services are up.

Setup Export Client
-------------------
Add the export client(s) as needed
{"name":"CloudMQTTEndpoint","addressable":{"name":"CloudMQTTEndpointAddress","protocol":"TCP","address":"m15.cloudmqtt.com","port":14866,"publisher":"edgex","user":"dtdsmcdu", "password":"password","topic":"data"},"format":"JSON","enable":true,"destination":"MQTT_TOPIC"}

Setup Rules
-----------
Then add the rules to fire the SNMP red patlite on MQTT temp > 100; set it off if < 101 (this has to be done each time because the rules are not being persisted on down of service)

{"name":"toohot", "condition": {"device":"MQTT test device","checks":[ {"parameter":"temp", "operand1":"Integer.parseInt(value)", "operation":">","operand2":"100" } ] }, "action" : {"device":"5c3a773c9f8fc20001e9fe70", "command":"5c3a773c9f8fc20001e9fe6a","body":"[{\\\"RedLightControlState\\\":\\\"2\\\",\\\"RedLightTimer\\\":\\\"0\\\"}]"},"log":"Patlite warning triggered for temp too high" }

{"name":"backnormal", "condition": {"device":"MQTT test device","checks":[ {"parameter":"temp", "operand1":"Integer.parseInt(value)", "operation":"<","operand2":"101" } ] }, "action" : {"device":"5c3a773c9f8fc20001e9fe70", "command":"5c3a773c9f8fc20001e9fe6a","body":"[{\\\"RedLightControlState\\\":\\\"1\\\",\\\"RedLightTimer\\\":\\\"0\\\"}]"},"log":"Patlite off" }

Add rules to fire the SNMP patlite on Modbus temp >= 200 set yellow light > 20 (and set green light off); if temp < 200 set green light on (and turn off yellow light)

{"name":"amberon", "condition": {"device":"cometThermostat1","checks":[ {"parameter":"Temperature", "operand1":"Integer.parseInt(value)", "operation":">","operand2":"200" } ] }, "action" : {"device":"5c3a773c9f8fc20001e9fe70", "command":"5c3a773c9f8fc20001e9fe6c","body":"[{\\\"AmberLightControlState\\\":\\\"2\\\",\\\"AmberLightTimer\\\":\\\"0\\\"}]"},"log":"Patlite Amber on" }

{"name":"greenoff", "condition": {"device":"cometThermostat1","checks":[ {"parameter":"Temperature", "operand1":"Integer.parseInt(value)", "operation":">","operand2":"200" } ] }, "action" : {"device":"5c3a773c9f8fc20001e9fe70", "command":"5c3a773c9f8fc20001e9fe6b","body":"[{\\\"GreenLightControlState\\\":\\\"1\\\",\\\"GreenLightTimer\\\":\\\"0\\\"}]"},"log":"Patlite Green off" }

{"name":"amberoff", "condition": {"device":"cometThermostat1","checks":[ {"parameter":"Temperature", "operand1":"Integer.parseInt(value)", "operation":"<=","operand2":"200" } ] }, "action" : {"device":"5c3a773c9f8fc20001e9fe70", "command":"5c3a773c9f8fc20001e9fe6c","body":"[{\\\"AmberLightControlState\\\":\\\"1\\\",\\\"AmberLightTimer\\\":\\\"0\\\"}]"},"log":"Patlite Amber off" }

{"name":"greenon", "condition": {"device":"cometThermostat1","checks":[ {"parameter":"Temperature", "operand1":"Integer.parseInt(value)", "operation":"<=","operand2":"200" } ] }, "action" : {"device":"5c3a773c9f8fc20001e9fe70", "command":"5c3a773c9f8fc20001e9fe6b","body":"[{\\\"GreenLightControlState\\\":\\\"2\\\",\\\"GreenLightTimer\\\":\\\"0\\\"}]"},"log":"Patlite Green on" }

Populate from MQTT
------------------
Submit to DataTopic: {"cmd":"temp","method":"get","name":"MQTT test device","temp":102}

Modbus Background
-----------------
Determine tty/usb port:  dmesg|grep tty

Make sure access to the port is open
	sudo chmod 666 /dev/ttyUSB0^C

install modpoll to test connection to Modbus device.  For the Comet thermostat
	sudo ./modpoll -m rtu -a 1 -c 1 -1 -b 9600 -d 8 -s 2 -p none -r 49 /dev/ttyUSB0

-m rtu Modbus RTU protocol (default if SERIALPORT contains /, \\ or COM)
-a # Slave address
-c # Number of values to poll
Poll only once only
-b # baud rate
-d # databits
-s # stopbits
-p none no parity
-r # start reference (start offset)

