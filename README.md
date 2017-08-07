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
* *Characteristic*: Each service has a set of properties called characteristics. The garage door has a `Current Door State` and a `Target Door State` boolean. Each characteritics of a service identifies its current state. Each characteristic has 3 permission levels : read, write and notify.

Homekit enables you to choose within a list of existing service, each associated with their compulsory characteristics. Any optional characteristics can be added to a service based on your needs.

**What if my object is not Homekit compatible**

You need a proxy between Homekit and your device. Most connected objects manufacturers develop their own API.
Your proxy will receive Homekit requests and translate them according to your object API and protocols.
The proxy used for this article is a wrapper of [HAP-node.js](https://github.com/KhaosT/HAP-NodeJS) called [Homebridge](https://github.com/nfarina/homebridge).

## Let's code

Our device is a switch. It can be controlled through a RESTful API over HTTP protocol on our LAN.

We will create a Homebridge plugin registering a new Accessory with two services:
* `AccessoryInformation` service, required for every accessory, whatever the type, broadcasting information related to the device itself
* `Switch` service, corresponding to our actual switch. Such service has a single `On` boolean characteristic.

First, we need to inject our plugin within homebridge.
`mySwitch` is the javascript object that will contain our control logic.

```javascript
var Service, Characteristic;

module.exports = function (homebridge) {
  Service = homebridge.hap.Service;
  Characteristic = homebridge.hap.Characteristic;
  homebridge.registerAccessory("switch-plugin", "MyAwesomeSwitch", mySwitch);
};
```

The core logic built within HAP-node.js and Homebridge is located wihtin the `getServices` prototype function of `mySwitch` object.
We will instanciate our services in this function. We will also define which getter and setter of each characteristic of each service it shall call on every requests received from Homekit.

We need to instanciate :
* an `AccessoryInformation` service containing:
  * a `Name` characteristic - the only required characteristic of this service
  * a `Manufacturer` characteristic
  * a `Model` characteristic
  * a `SerialNumber` characterisic
* a `Switch` service containing:
  * an `On`characteristic - the only required characteristic of this service
  
Unlike `AccessoryInformation` service's characteristics, which are readable and can be set at plugin initialisation, the `On` characteristic is writtable and require a getter and setter. 

```javascript
mySwitch.prototype = {
  getServices: function () {
    var informationService = new Service.AccessoryInformation();
    informationService
      .setCharacteristic(Characteristic.Name, "My switch")
      .setCharacteristic(Characteristic.Manufacturer, "My switch manufacturer")
      .setCharacteristic(Characteristic.Model, "My switch model")
      .setCharacteristic(Characteristic.SerialNumber, "123-456-789");

    var switchService = new Service.Switch("My switch");
    switchService
      .getCharacteristic(Characteristic.On)
        .on('get', this.getSwitchOnCharacteristic.bind(this))
        .on('set', this.setSwitchOnCharacteristic.bind(this));

    this.informationService = informationService;
    this.switchService = switchService;
    return [informationService, switchService];
  }
};
```

We will now write the logic of `On` characteristic getter and setter within dedicated prototype function of `mySwitch` object.
We will make the following assumption regarding the RESTful API offered by the switch :
* GET requests on http://<mySwitchIp>/api/status returns a `{ currentState: <boolean> }` reflecting the switch current state
* POST requests on http://<mySwitchIp>/api/order sending a `{ targetState: <boolean> }` reflecting desired target state set the switch state

We will use `request` and `url` modules to peform our HTTP requests.

Our configuration object, defined within Homebridge global configuration JSON, will contain both URLs described above.

```javascript
const request = require('request');
const url = require('url');

function mySwitch(log, config) {
  this.log = log;
  this.getUrl = url.parse(config['getUrl']);
  this.postUrl = url.parse(config['postUrl']);
}

mySwitch.prototype = {

  getSwitchOnCharacteristic: function(next) {
    var me = this;
    request({
        url: me.getUrl,
        method: 'GET',
    }, 
    function (error, response, body) {
      if (error) {
        me.log('STATUS: ' + response.statusCode);
        me.log(error.message);
        next(error);
      }
      next(null, body.currentState);
    });
  },
  
  setSwitchOnCharacteristic: function(on, next) {
    var me = this;
    request({
      url: me.postUrl,
      body: {'targetState': on},
      method: 'POST',
      headers: {'Content-type': 'application/json'}
    },
    function (error, response) {
      if (error) {
        me.log('STATUS: ' + response.statusCode);
        me.log(error.message);
        next(error);
      }
      next();
  }
```

