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

- docker build -t jims/docker-device-snmp-patlite:0.9.0 .
- docker build -t jims/docker-device-mqtt:0.9.0 .
- docker build -t jims/docker-edgex-ui-clojure:0.9.0 .

Run
---
Then docker compose up the services.  
Note that rules engine starts slowly and may have to be brought up after all other services are up.

Setup and Populate
------------------
Add the export client(s) as needed
{"name":"CloudMQTTEndpoint","addressable":{"name":"CloudMQTTEndpointAddress","protocol":"TCP","address":"m15.cloudmqtt.com","port":14866,"publisher":"edgex","user":"dtdsmcdu", "password":"password","topic":"data"},"format":"JSON","enable":true,"destination":"MQTT_TOPIC"}

Then add the rules to fire the SNMP patlite on temp > 100; set it off if < 101 (this has to be done each time because the rules are not being persisted on down of service)

{"name":"toohot", "condition": {"device":"MQTT test device","checks":[ {"parameter":"temp", "operand1":"Integer.parseInt(value)", "operation":">","operand2":"100" } ] }, "action" : {"device":"5c26c1eb9f8fc20001eee870", "command":"5c26c1eb9f8fc20001eee86a","body":"[{\\\"RedLightControlState\\\":\\\"2\\\",\\\"RedLightTimer\\\":\\\"0\\\"}]"},"log":"Patlite warning triggered for temp too high" }

{"name":"backnormal", "condition": {"device":"MQTT test device","checks":[ {"parameter":"temp", "operand1":"Integer.parseInt(value)", "operation":"<","operand2":"101" } ] }, "action" : {"device":"5c26c1eb9f8fc20001eee870", "command":"5c26c1eb9f8fc20001eee86a","body":"[{\\\"RedLightControlState\\\":\\\"1\\\",\\\"RedLightTimer\\\":\\\"0\\\"}]"},"log":"Patlite off" }

Then submit to DataTopic: {"cmd":"temp","method":"get","name":"MQTT test device","temp":102}

