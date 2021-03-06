# unifi-axios-events 

[![License][mit-badge]][mit-url]

unifi-axios-events is a Node.js module that allows you to listen for events from and call methods on the UniFi API (UniFi is Ubiquiti Networks wifi controller software). It is heavily derivitive of [unifi-events](https://github.com/oznu/unifi-events). I was having issues with it interfacing with my UDM Pro and ended up porting the code to Axios since request is now deprecated. I should work with any Unifi controller but I have only tested it with the UDM Pro.

## Requirements

* Node.js v6 or later
* [UniFi-Controller](https://www.ubnt.com/download/unifi) v5

## Installation

`$ npm install unifi-axios-events`

## Example

```javascript
const Unifi = require('unifi-axios-events')

const unifi = new Unifi({
  host: 'unifi',                        // The hostname or ip address of the unifi controller (default: 'unifi')
  port: 8443,                           // Port of the unifi controller (default: 8443)
  username: 'admin',                    // Username (default: 'admin').
  password: 'ubnt',                     // Password (default: 'ubnt').
  site: 'default',                      // The UniFi site to connect to (default: 'default').
  insecure: true,                       // Allow connections if SSL certificate check fails (default: false).
});

unifi.init();

// Listen for any event
unifi.on('**', function (data) {
  console.log(this.event, data);
});
```

## Events

unifi-events uses [EventEmitter2](https://github.com/asyncly/EventEmitter2) and namespaced events. 

### namespace `ctrl`

These events indicate the status of the connection to the UniFi controller

* `ctrl.connect` - emitted when the connection to the controller is established
* `ctrl.disconnect` - emitted when the connection to the controller is lost
* `ctrl.error` - 
* `ctrl.reconnect` - 


### namespaces `wu`, `wg`, `lu`, ...

This JSON file shows all possible events: https://demo.ubnt.com/manage/locales/en/eventStrings.json?v=5.4.11.2
The prefix `EVT_` gets stripped, the first underscore is replaced by the namespace separating dot, everything is 
converted to lower case. Some events such as `EVT_AD_LOGIN` (Admin Login) are not emitted by the UniFi Controller.


#### Example Wireless User events

* `wu.connected` - Wireless User connected
* `wu.disconnected` - Wireless User disconnected
* `wu.roam` - Wireless User roamed from one AP to another
* `wu.roam_radio` - Wireless User changed channel on the same AP

#### Example Wireless Guest Events

* `wg.connected` - Wireless Guest connected
* `wg.disconnected` - Wireless Guest disconnected
* `wg.roam` - Wireless Guest roamed from one AP to another
* `wg.roam_radio` - Wireless Guest changed channel on the same AP
* `wg.authorization_ended` - Wireless Guest became unauthorised

#### Wildcard usage

Example listing for events on Guest Wireless networks only:

```javascript
unifi.on('wg.*', function (data) {
  console.log(this.event, data);
})
```

Example listening for connected events on all network types:

```javascript
unifi.on('*.connected', function (data) {
  console.log(this.event, data);
});
```

## Methods

#### init()

Initialize the connection to the UniFi controller. Must be called after the constructor.

#### connect()

Connect to the UniFi controller. Is called by init(), so normally you don't need to call it (except if you want 
to re-establish a connection that was closed before).

#### close()

Closes the connection to the UniFi controller


### UniFi API Methods

Following methods operate on the configured site. The path gets prefixed with 
`https://<host>:<port>/api/s/<site>/` if it does not start with a slash, otherwise it gets prefixed with 
`https://<host>:<port>`. To explore available API endpoints you can use the 
[UniFi-API-browser](https://github.com/Art-of-WiFi/UniFi-API-browser).

These methods are returning a promise.


#### get(path)

Do a HTTP GET on the API.

**Examples:**

* Get a list of all clients
```javascript
unifi.get('stat/sta').then(console.log);
```

* Get infos of a specific client
```javascript
unifi.get('stat/user/<mac>').then(console.log);
```

* Get alarms
```javascript
unifi.get('list/alarm').then(console.log);
```

* Get wireless network IDs
```javascript
unifi.get('rest/wlanconf').then(res => {
    res.data.forEach(wlan => {
        console.log(wlan.name, wlan._id);
    });
});
```

* Get device IDs
```javascript
unifi.get('stat/device').then(res => {
     res.data.forEach(dev => {
        console.log(dev.name, dev._id);
     });
});
```

#### del(path)

Do a HTTP DELETE on the API.

#### post(path, body)

Do a HTTP POST on the API.

**Examples:**

* Enable all LEDs of all APs
```javascript
unifi.post('set/setting/mgmt', {led_enabled: true}).then(console.log);
```

* Disable a WLAN
```javascript
unifi.post('upd/wlanconf/<wlan_id>', {enabled: false}).then(console.log);
```

#### put(path, body)

Do a HTTP PUT on the API.

**Examples:**

* Enable LED of AP
```javascript
unifi.put('rest/device/<device_id>', {led_override: 'on'}).then(console.log);
```


## License

* MIT © 2021 woogie (https://github.com/worldwidewoogie)
* MIT © 2017-2021 [oznu](https://github.com/oznu)
* MIT © 2018 [Sebastian Raff](https://github.com/hobbyquaker)    

[mit-badge]: https://img.shields.io/badge/License-MIT-blue.svg?style=flat
[mit-url]: LICENSE
