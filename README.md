# Miio Device Library

This is a small library for controlling Mi Home devices that implement the Miio
protocol, such as the Mi Air Purifier.

## Installation

```
npm install --save miio
```

## Usage

```javascript
const miio = require('miio');
```

Create a handle to the device:

```javascript
// Create a device - with a model to get a more specific device instance
const device = miio.createDevice({
	model: 'zhimi-airpurifier-m1',
	address: '192.168.100.8'
});

// Create a new generic device by skipping the model
const device = miio.createDevice({
	address: '192.168.100.8'
});
```

Call methods to interact with the device:

```javascript
// Call any method via call
device.call('set_mode', [ 'silent' ])
	.then(console.log)
	.catch(console.error);

// Or use some of the built-in methods
device.setPower(false);

device.getProperties([ 'power', 'mode', 'aqi', 'temp_dec', 'humidity' ])
	.then(console.log)
	.catch(console.error);
```

Monitor and transform properties:
```javascript
// Define a property that should be monitored
device.defineProperty('mode');

// Define that a certain property should be run through a custom conversion
device.defineProperty('temp_dec', v => v / 10.0);

// Listen for changes to properties
device.on('propertyChanged', e => console.log(e.property, e.oldValue, e.value));

// Activate automatic property monitoring (activated by default for most devices)
device.monitor();

// Stop automatic property monitoring
device.stopMonitoring();

// Fetch the last value of a monitored property
const value = device.property('temp_dec');
```

## Discovering devices

This library does not perform discovery of devices, but can be combined with
mDNS discovery to find Miio-compatible devices. Devices announce themselves
via `_miio._udp` and should work for most devices, in certain cases you might
need to restart your device to make it announce itself.

Here is an example on how to find devices with `tinkerhub-mdns`;

```javascript
const browser = require('tinkerhub-mdns')
	.browser({
		type: 'miio',
		protocol: 'udp'
	});

browser.on('available', reg => {
	// Use infoFromHostname to figure out model and id of device
	const info = miio.infoFromHostname(reg.name);
	if(! info) return;

	// Set the address and port from the found registration
	info.address = reg.addresses[0];
	info.port = reg.port;

	const device = miio.createDevice(info);

	// Do something with the device here
});
```

# Devices types

The intent of this library is to encompass all Miio-compatible devices and to
provide and easy to use API for them. To do this types are mapped from their
model name into specific device types that provide a friendly API to the device.

Currently implemented devices are:

* Air Purifier
* Smart Socket Plug

See [documentation for devices](devices.md) for information about the API for each
type.

# Protocol documentation

This library is based on the documentation provided by OpenMiHome. See https://github.com/OpenMiHome/mihome-binary-protocol for details.

# Missing features

* Sub devices on a Lumi (Mi Home Gateway) are not supported
* A lot of specific device types are missing