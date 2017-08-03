# How to make Siri your perfect home companion without Apple Homekit compatible devices

## Why Homekit ?

Homekit is a home accessory management framework developped by Apple. 
It allows Apple devices' owners to control connected objects from different manufacturers using a single interface.
It enhances Siri's capability to interprete commands intended for those devices.

## How doest it work ?

As of today, only a small number of Homekit enabled devices are available on the market.
The software included within those devices is able to receive requests made using Homekit protocol.

**What if my object is not Homekit compatible**

You need a proxy between Homekit and your device. Most connected objects manufacturers develop their own API.
Your proxy will receive Homekit requests and translate them according to your object API and protocols.

Our device is a switch. It can be controlled through a RESTful API over HTTP protocol on our LAN.
