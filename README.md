thingamabob.js - Control every _thing_
-------------------------------------------------------------------------------

Thingamabob is a universal library for things.  It provides a standardized way
to interact with smart devices made by various vendors.

Initially, support is planned for:
    - Philips Hue
    - All Canopy-enabled devices

Initialization
------------------------------------------------------------------------------

Add the following to your HTML to include the library:

    <script src="thingamabob.js"></script>

Depending on the vendors & services you need to interact with, also include:

    <script src="thingamabob-service-philips.js"></script>
    <script src="thingamabob-service-canopy.js"></script>

To initialize, use:

    var bob = ThingamabobInit();

By convention, we call the library context `bob`.

You can now tell `bob` about the services that it should talk to using
`addService`.

To interact with Philips devices:

    bob.addService({
        serviceType: "philips",
        host: "192.168.1.10",    /* IP address of base station */
        username: "22a828f1898a4257c3f181e753241337"
        onSuccess: function({
            alert("connected!");
        }),
        onError: function({
            alert("oops!");
        });
    });

To interact with Canopy-enabled devices:

    bob.addService({
        serviceType: "canopy",
        host: "canopy.link",    /* Hostname of canopy server */
        username: "myUsername",
        password: "myPassword",
        onSuccess: function({
            alert("connected!");
        }),
        onError: function({
            alert("oops!");
        });
    });

You can add as many services as you'd like.  Each time addService is
successfully called, it adds more devices to the `bob` object.

Selecting Devices
------------------------------------------------------------------------------

The method `bob.devices()` returns a *DeviceList* object.  A *DeviceList* can be
used to select devices based on a number of criteria.  Here are some examples:

    var all = bob.devices();
    var connected = bob.devices(":connected");
    var disconnected = bob.devices(":disconnected");
    var deviceByID = bob.devices("%canopy:a9430fcde9f103a1ffee90");
    var devicesByName = bob.devices("#My Device");
    var hueLights = bob.devices("@philips.hue");
    var filtered = bob.devices().filter({
        connected: true,
        interface: "philips.hue",
    };

Each of the above examples returns a *DeviceList* object containing a
filtered set of devices.

You can also select all devices according to their capabilities.  Every device
satisfies one or more "interfaces".  For example, the Philips Hue satisfies the
following interfaces:

    philips.hue
    std.onOff              // it has "on" property
    std.light.dimmable     // it has "brightness" property

Using the `@` symbol in `bob.devices` lets you select devices that implement a
particular interface.

    var philipsLightBulbs = bob.devices("@philips.hue");
    var dimmables = bob.devices("@std.light.dimmable");

Each *DeviceList* contains 0 or more *Device* objects, that can be accessed
using array notation.  You can loop over the devices in a *DeviceList* like so:

    var hueLights = bob.devices("@philips.hue");
    for (var i = 0; i < hueLights.length i++) {
        var light = hueLights[i];
        console.log(light.name());
        console.log(light.brightness());
    }


Getting Device Information
------------------------------------------------------------------------------
The *Device* object has methods for getting information and status about a
device:

    myDevice = bob.devices()[0];
    if (myDevice) {
        var name = myDevice.name();
        // "Attic Light"

        var id = myDevice.id();
        // "philips:18"

        var interfaces = myDevice.intefaces()
        // ["philips.hue", "std.onOff", "std.light.dimmable"]

        var connected = myDevice.connected()
        // true

        var props = myDevice.properties()
        // <PropertyList Object>
    }

Controlling Devices
------------------------------------------------------------------------------

You can control devices with `.set(...)`.  This can be used on a single
*Device* object, or on a *DeviceList* object for bulk updates.

    // Turn off all devices.
    // This instruction will be ignored by devices that don't have an "on"
    // property.
    bob.devices().set({
        on: false
    });

    // Turn on a particular light and set its brightness to max.
    var myDevice = bob.devices("#Attic Light")[0];
    if (myDevice) {
        myDevice.set({
            on: True,
            brightness: 1.0,
        });
    }
