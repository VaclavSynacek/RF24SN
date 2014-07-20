RF24SN
======

Connects sensors/actuators to MQTT broker.
Simpler than MQTT-SN. Runs on nRF24L01 hardware.

# Overview

![Overview](https://raw.githubusercontent.com/VaclavSynacek/RF24SN/master/RF24SN.png "Overview")

RF24SN consists of client and server implementations. Client implementations
run on Arduino compatible microcontrollers. They can easily publish measured
sensor values to MQTT broker or request values from MQTT broker to be used in actuators. The server
implementations run on TCP/IP connected devices and perform translation
from RF24SN protocol to MQTT protocol - they act as gateways between the
relatively dumb clients and MQTT broker.

## RF24SN Client

RF24SN Client is a library for Arduino compatible microcontrollers.

Publish Usage:
```Arduino
#define SENSOR_ID 1  //publishes to RF24SN/in/<nodeId>/1
       
float measuredValue = 3.14; //replace this line with actual sensor reading code
bool publishSuccess = rf24sn.publish(SENSOR_ID, measuredValue);
```

Request Usage:
```Arduino
#define SENSOR_ID 2  //request last payload fo RF24SN/out/<nodeId>/2
       
float requestValue;
bool requestSuccess = rf24sn.request(SENSOR_ID, &requestValue);
if(requestSuccess)
{
   Serial.println(requestValue); //replace this line with some action code (moving servo, LED on, etc.)
}
```

For more details see [its own repository](https://github.com/VaclavSynacek/RF24SN_Arduino_Client).


## RF24SN Server

RF24SN Server software can be generally used as-is with only
minimal configuration. It's sole purpose is to act as a gateway between RF24SN network and MQTT broker. It should be able to connect to any MQTT compatible broker - currently only [mosquitto](http://mosquitto.org/)
has been tested.

### RF24SN on nodejs using node-nrf driver

Full featured implementation with little dependencies. Should run on all
platforms where there is nodejs and the [node-nrf](https://github.com/natevw/node-nrf) driver / [pi-spi](https://github.com/natevw/pi-spi) driver - currently it has been tested on Raspberry Pi.

Usage:
```Shell
sudo rf24sn -b mqtt://localhost:1883 -spi /dev/spidev0.0 -ce 25 -irq 24
```

For more details see [its own repository](https://github.com/VaclavSynacek/RF24SN_nodejs_Server). 


### RF24SN in C++ using RF24 driver
Implementation uses [RF24-rpi](https://github.com/jscrane/RF24-rpi) - a Raspberry Pi port of arduino [RF24](https://maniacbug.github.io/RF24/index.html) driver.
Currently only runs on Raspberry Pi.
Implementation is incomplete - the gateway communicates with RF24SN clients,
but is not connected to MQTT broker (only prints received communication
to standard out).

Usage:
```Shell
sudo RF24SN
```

For more details see [its own repository](https://github.com/VaclavSynacek/RF24SN_CPP_Server).

### RF24SN Spark Core Server
Not yet implemented.

For more details see [its own repository](https://github.com/VaclavSynacek/RF24SN_Spark_Core_Server).

# Compared to alternatives

RF24SN use cases are very similar to the following solutions,
but with the following diferencies:

## MQTT-S(N)

Both [MQTT-S](http://mqtt.org/MQTT-S_spec_v1.2.pdf)
and RF24SN are meant to be MQTT simplifications for sensor
networks with sleepy nodes communicating over unreliable radios.
The differences are:

* MQTT-S is a standard.
* MQTT-S is closer to MQTT.
* MQTT-S supports publish and subscribe to any MQTT topic;
RF24SN supports publish and request only on predefined topic prefixes
* MQTT-S implements subscription in MQTT sense, the gateway pushes MQTT
messages to nodes when they wake up;
RF24SN implements topic polling, nodes query for last known messages
on a topics rather than waiting for the push.
* RF24SN is more bit eficient.
* RF24SN is much easier to implement
* MQTT-S is agnostic of underlying radio tecnology, however most
implementations only support ZigBee;
RF24SN currently only suports nRF24l01 radio chips.

## RFXduino

Both [RFXduino](http://embeddedcoolness.com)
and RF24SN implement client and server/gateway
software for nRF24L01 radio networks.
The differencies are:

* RFXduino is open source and free for non-commercial use;
RF24SN is open source and truly free as in speach (MIT license).
* RFXduino suports all kinds of added logic such as email sending,
SMS, shell command execution; RF24SN only supports publishing of and
requests for individual values.
* RF24SN is much easier to implement

## RF24Network

Both [RF24Network](https://maniacbug.github.io/RF24Network/)
and RF24SN implement logic on top of [RF24](https://maniacbug.github.io/RF24/index.html)
driver, however with the following differencies:

* RF24Network follows tree topology with each node capable of communication
with 6 children nodes and 1 parent node;
RF24SN follows star topology with the base capable of communication with
254 children nodes.
* RF24Network supports transmission of any payload up to 32 bytes;
RF24SN only supports publishing of and requests for individual float values.

