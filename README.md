# bleno

[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/sandeepmistry/bleno?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


A Node.js module for implementing BLE (Bluetooth Low Energy) peripherals.

Need a BLE central module? See [noble](https://github.com/sandeepmistry/noble).


## Prerequisites

### Windows

 * [node-gyp requirements for Windows](https://github.com/TooTallNate/node-gyp#installation)
   * Python 2.7
   * Visual Studio ([Express](https://www.visualstudio.com/en-us/products/visual-studio-express-vs.aspx))
 * [node-bluetooth-hci-socket prerequisites](https://github.com/sandeepmistry/node-bluetooth-hci-socket#windows)
   * Compatible Bluetooth 4.0 USB adapter
   * [WinUSB](https://msdn.microsoft.com/en-ca/library/windows/hardware/ff540196(v=vs.85).aspx) driver setup for Bluetooth 4.0 USB adapter, using [Zadig tool](http://zadig.akeo.ie/)

## Install

```sh
npm install bleno
```

## Usage

```javascript
var bleno = require('bleno');
```

See [examples folder](https://github.com/sandeepmistry/bleno/blob/master/examples) for code examples.

### Actions

#### Advertising

##### Start advertising

NOTE: ```bleno.state``` must be ```poweredOn``` before advertising is started. ```bleno.on('stateChange', callback(state));``` can be used register for state change events.

```javascript
var name = 'name';
var serviceUuids = ['fffffffffffffffffffffffffffffff0']

bleno.startAdvertising(name, serviceUuids[, callback(error)]);
```

 __Note:__: there are limits on the name and service UUID's

  * name
    * maximum 26 bytes
  * service UUID's
    * 1 128-bit service UUID
    * 1 128-bit service UUID + 2 16-bit service UUID's
    * 7 16-bit service UUID


##### Start advertising iBeacon

```javascript
var uuid = 'e2c56db5dffb48d2b060d0f5a71096e0';
var major = 0; // 0x0000 - 0xffff
var minor = 0; // 0x0000 - 0xffff
var measuredPower = -59; // -128 - 127

bleno.startAdvertisingIBeacon(uuid, major, minor, measuredPower[, callback(error)]);
```

```javascript
var scanData = new Buffer(...); // maximum 31 bytes
var advertisementData = new Buffer(...); // maximum 31 bytes

bleno.startAdvertisingWithEIRData(advertisementData[, scanData, callback(error)]);
```

  * For EIR format section [Bluetooth Core Specification](https://www.bluetooth.org/docman/handlers/downloaddoc.ashx?doc_id=229737) sections and 8 and 18 for more information the data format.

##### Stop advertising

```javascript
bleno.stopAdvertising([callback]);
```

#### Set services

Set the primary services available on the peripheral.

```javascript
var services = [
   ... // see PrimaryService for data type
];

bleno.setServices(services[, callback(error)]);
```

#### Disconnect client

```javascript
bleno.disconnect(); // Linux only
```

#### Update RSSI

```javascript
bleno.updateRssi([callback(error, rssi)]); // not available in OS X 10.9
```

### Primary Service

```javascript
var PrimaryService = bleno.PrimaryService;

var primaryService = new PrimaryService({
    uuid: 'fffffffffffffffffffffffffffffff0', // or 'fff0' for 16-bit
    characteristics: [
        // see Characteristic for data type
    ]
});
```

### Characteristic

```javascript
var Characteristic = bleno.Characteristic;

var characteristic = new Characteristic({
    uuid: 'fffffffffffffffffffffffffffffff1', // or 'fff1' for 16-bit
    properties: [ ... ], // can be a combination of 'read', 'write', 'writeWithoutResponse', 'notify', 'indicate'
    secure: [ ... ], // enable security for properties, can be a combination of 'read', 'write', 'writeWithoutResponse', 'notify', 'indicate'
    value: null, // optional static value, must be of type Buffer - for read only characteristics
    descriptors: [
        // see Descriptor for data type
    ],
    onReadRequest: null, // optional read request handler, function(offset, callback) { ... }
    onWriteRequest: null, // optional write request handler, function(data, offset, withoutResponse, callback) { ...}
    onSubscribe: null, // optional notify/indicate subscribe handler, function(maxValueSize, updateValueCallback) { ...}
    onUnsubscribe: null, // optional notify/indicate unsubscribe handler, function() { ...}
    onNotify: null, // optional notify sent handler, function() { ...}
    onIndicate: null // optional indicate confirmation received handler, function() { ...}
});
```

#### Result codes

  * Characteristic.RESULT_SUCCESS
  * Characteristic.RESULT_INVALID_OFFSET
  * Characteristic.RESULT_INVALID_ATTRIBUTE_LENGTH
  * Characteristic.RESULT_UNLIKELY_ERROR

#### Read requests

Can specify read request handler via constructor options or by extending Characteristic and overriding onReadRequest.

Parameters to handler are
  * ```offset``` (0x0000 - 0xffff)
  * ```callback```


```callback``` must be called with result and data (of type ```Buffer```) - can be async.

```javascript
var result = Characteristic.RESULT_SUCCESS;
var data = new Buffer( ... );

callback(result, data);
```

#### Write requests

Can specify write request handler via constructor options or by extending Characteristic and overriding onWriteRequest.

Parameters to handler are
  * ```data``` (Buffer)
  * ```offset``` (0x0000 - 0xffff)
  * ```withoutResponse``` (true | false)
  * ```callback```.

```callback``` must be called with result code - can be async.

```javascript
var result = Characteristic.RESULT_SUCCESS;

callback(result);
```

#### Notify subscribe

Can specify notify subscribe handler via constructor options or by extending Characteristic and overriding onSubscribe.

Parameters to handler are
  * ```maxValueSize``` (maximum data size)
  * ```updateValueCallback``` (callback to call when value has changed)

#### Notify unsubscribe

Can specify notify unsubscribe handler via constructor options or by extending Characteristic and overriding onUnsubscribe.

#### Notify value changes

Call the ```updateValueCallback``` callback (see Notify subscribe), with an argument of type ```Buffer```

Can specify notify sent handler via constructor options or by extending Characteristic and overriding onNotify.

### Descriptor

```javascript
var Descriptor = bleno.Descriptor;

var descriptor = new Descriptor({
    uuid: '2901',
    value: 'value' // static value, must be of type Buffer or string if set
});
```
