# How to make Siri your perfect home companion without Apple Homekit compatible devices

## Why Homekit ?

Homekit is a home accessory management framework developped by Apple. 
It allows Apple devices' owners to control connected objects from different manufacturers using a single interface.
It enhances Siri's capability to interprete commands intended for those devices.

## How doest it work ?

As of today, only a small number of Homekit enabled devices are available on the market.
The software included within those devices is able to receive requests made using Homekit protocol.

**Homekit architecture**
* *Accessory*: An accessory can be associated to a physical object.
* *Service*: A service correspond to an object's function. A garage door may have a service to open and close the door as well as another service to turn on and off the garage light.
* *Characteristic*: Each service has a set of properties called characteristics. The garage door has a `Current Door State` and a `Target Door State` boolean. Each characteritics of a service identifies its current state.

Homekit enables you to choose within a list of existing service, each associated with their compulsory characteristics. Any optional characteristics can be added to a service based on your needs.

**What if my object is not Homekit compatible**

You need a proxy between Homekit and your device. Most connected objects manufacturers develop their own API.
Your proxy will receive Homekit requests and translate them according to your object API and protocols.
The proxy used for this article is a wrapper of [HAP-node.js](https://github.com/KhaosT/HAP-NodeJS) called [Homebridge](https://github.com/nfarina/homebridge).

## Let's code

Our device is a switch. It can be controlled through a RESTful API over HTTP protocol on our LAN.
We will create a Homebridge plugin registering a new Accessory with a single `Switch` service.
Such service has a single `On` boolean characteristic.

First, we need to provide inject our plugin within homebridge.
`mySwitch` is the javascript object that will contain our control logic.

```javascript
var Service, Characteristic;

module.exports = function (homebridge) {
  Service = homebridge.hap.Service;
  Characteristic = homebridge.hap.Characteristic;
  homebridge.registerAccessory("switch-plugin", "MyAwesomeSwitch", mySwitch);
};
```


