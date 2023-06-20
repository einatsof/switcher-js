# switcher-js2

*Fork of [@johnathanvidu JS implementation](https://github.com/johnathanvidu/switcher-js)*

switcher-js is a native nodejs library for controling [Switcher](https://switcher.co.il/)  smart home accessories - water heater, sockets, and blinds.<br/><br/>
It is a native javascript port of a wonderful python script (can be found [here](https://github.com/NightRang3r/Switcher-V2-Python)) created as a result of the extensive work which has been done by Aviad Golan ([@AviadGolan](https://twitter.com/AviadGolan)) and Shai rod ([@NightRang3r](https://twitter.com/NightRang3r)).

It is a work in progress and there is still a lot of work left to do.

I built it according to my specific needs and my specific device. If any issue arises, please feel free to open an issue and I'll do my best to help.

Current supported devices known to work with switcher-js:

- **Switcher Lights SL03**
- **Switcher Lights SL02**
- **Switcher Lights SL01**
- **Switcher Runner S12**
- **Switcher Runner S11**
- **Switcher Runner Mini**
- **Switcher Runner**
- **Switcher V4**
- **Switcher Mini**
- **Switcher V3**: (Switcher touch) - Firmware **V1.51**
- **Switcher V2**: Firmware **3.21** (Based on ESP chipset) 
- **Switcher V2**: Firmware**72.32** (Qualcomm chipset)

## Installation
Use [npm](https://www.npmjs.com/) to install switcher-js.
```bash
npm install switcher-js2
```

## Usage:
First, import the Switcher class from the switcher-js2 module:
```javascript
const Switcher = require('switcher-js2');
```
The Switcher class represents a switcher device and provides methods to interact with it. To create a new instance of the Switcher class, use the following constructor:
```javascript
let switcher = new Switcher(deviceID, deviceIP, logFunction, listenEnabled, deviceType);
```
* `deviceID` (string): The identifier of the switcher device.
* `deviceIP` (string): The IP address of the switcher device.
* `logFunction` (function): A callback function to handle log messages.
* `listenEnabled` (boolean): Specifies whether listening is enabled for device status messages.
* `deviceType` (string): The type of the switcher device.

### Discover
#### discover(logFunction[, deviceIdentifier[, discoveryTimeout]])
Discovers switcher devices on the network.
* `logFunction` (required): A callback function that handles log messages.
* `deviceIdentifier` (optional): A specific device identifier used to filter the discovery process. It could be Switcher name, IP or device-id. Only devices matching the provided identifier will be included in the result. If not specified, all available devices will be returned.
* `discoveryTimeout` (optional): The duration (in seconds) for which the discovery process should run.

discover will emit a ready event when auto discovery completed.

To use the auto discover functionallity use: 
```javascript
const Switcher = require('switcher-js2');

let proxy = Switcher.discover(console.log);

proxy.on('ready', (switcher) => {
    switcher.turn_on(); // switcher is a new initialized instance of Switcher class
});


setTimeout(() => {
    proxy.close(); // optional way to close the discovery (if discoveryTimeout is not set)
}, 10000);

```


### Control

```javascript
const Switcher = require('switcher-js2');

let switcher = new Switcher(deviceID, deviceIP, console.log, true, deviceType);

// status broadcast message - only works when listenEnabled is true
switcher.on('status', (status) => { 
    console.log(status)
    /* response:
    {
        power: 1,
        remaining_seconds: 591,
        default_shutdown_seconds: 5400,
        power_consumption: 2447 // in watts
    }
    */
});
switcher.on('state', (state) => { // state is the new switcher state
    console.log(state) // 1
});
switcher.on('error', (error) => {

});

switcher.turn_on();   // turns switcher on
switcher.turn_on(15); // turns switcher on for 15 minutes
switcher.turn_off();  // turns switcher off
switcher.set_default_shutdown(3600) // set the default auto shutdown to 1 hour (must be between 3600 and 86340)
switcher.status(status => { // get status
    console.log(status);
});
switcher.close();     // closes any dangling connections safely
```

### Control Runner Devices (blinds)

```javascript
const Switcher = require('switcher-js2');

let runner = new Switcher(deviceID, deviceIP, logFunction, listenEnabled, 'runner'); 
// set 'device-type' to 'runner' if you want to control the runner devices

runner.on('status', (status) => { // status broadcast message - only works when listenEnabled=true
    console.log(status)
    /* response:
    {
        position: 80,
        direction: 'STOP'
    }
    */
});
runner.on('position', (pos) => { // position is the new switcher position
    console.log(pos) // 100
});
switcher.on('error', (error) => {

});

switcher.set_position(80);   // Set blinds position to 80%

switcher.stop_runner() // stop the blinds

switcher.close();     // closes any dangling connections safely
```

### Listen
#### listen(logFunction[, deviceIdentifier])
A global listen functionality that listens to a single or multiple switcher devices for status messages.
* `logFunction` (required): A callback function that handles log messages.
* `deviceIdentifier` (optional) - you can provide the Switcher name, IP or device-id to filter specific device messages.

To use the listen functionallity use: 
```javascript
const Switcher = require('switcher-js2');

let proxy = Switcher.listen(console.log);

proxy.on('message', (message) => {
    console.log(message)
    /* response:
    {
        device_id: 'e3a845',
        device_ip: '10.0.0.1',
        name: 'Boiler',
        type: 'v4'
        state: {
            power: 1,
            remaining_seconds: 591,
            default_shutdown_seconds: 5400,
            power_consumption: 2447 // in watts
        }
    }
    */
});

proxy.close(); // close the listener socket

```

proxy will emit a message event every time it receives a message from a switcher device.

## Multiple Connections

Don't use Discover, Listen and Switcher with (listenEnabled=true) at the same time as it will return error since this socket is being used.
If you want to listen to multiple devices, use the global listen function to get all statuses, and use the switcher instance without the listen capability.

## Contributing
Pull requests are more than welcome. For major changes, please open an issue first to discuss what you would like to change.
Even coding tips and standards are welcome, I have very limited experience with javascript, so there's a lot of things I don't know are cleaner or more standarized in the industry.

## License
[MIT](https://choosealicense.com/licenses/mit/)