# Obelisk

Obelisk is a framework written in C designed for embedded systems to allow multiple modules with potential blocking i/o operations to run on a single binary in a coordinated way.

When compiling obelisk, a set of modules are selected. Those modules will be available to be instantiated by obelisk at run time.

Examples of typical modules available are _socketcan_ (to read and write to a CAN bus), _serial_ (to read and write to a serial port), _3g_ (to connect to internet with a 3G modem)...

## Instances

When starting, obelisk reads its config from a JSON file. Apart from giving some global details for obelisk (like the verbosity of the logs), it defines each instance of the modules that should be instantiated and the config for each instance.

A module can be instantiated multiple times, it's important to distinguish between a module (for example _socketcan_) and the instances of that module (for example *socketcan_0*, *socketcan_1*...). In obelisk, interaction is done with instances, not modules, as modules are only "templates" for instances to be created.

An example of valid JSON config that creates an instance (`instance_name`) of the module `some_module` is:

    {
        "main": {
            "log_level": "info"
        },
        "instance_name": {
            "module": "some_module",
            "log_level": "debug",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "some_module_param_1": "some_value",
            "some_module_param_2": true
        },
        ... other instances here ...
    }

As can be seen the configuration for each instance includes the following parameters:

* **Name:** Unique name for the instance (e.g. `instance_name`), used to refer to that instance.
* **Module:** Name of the module to instance (e.g. `some_module`). The type of instance that will be created.
* **Log level:** Log verbosity of the instance (one of: `"debug"`, `"info"`, `"notice"`, `"warning"`, `"error"`, `"crit"`, `"alert"`, `"emerg"` or `"none"`; by default `"none"` that means to take the value of the main log level).
* **Autostart:** Whether to start the instance automatically when starting obelisk (`true` by default).
* **Respawn:** Whether to restart the instance when it has stopped by an error (not when stopped by the user, `true` by default).
* **Maximum respawn delay:** The maximum waiting time between instance respawns (the respawn delay increments exponentially from 1 seconds until the defined maximum, `60` seconds by default).
* **Other module specific parameters:** Each type of module defines its own parameters.

The `main` section of the configuration holds the parameters for obelisk itself.

## The pubsub system

Obelisk and all the modules it instantiates are decoupled from each other and communicate through a pubsub system. 

### Features

* Listen to system-wide or instance-specific events to receive data from them or react to their state changes.
* Interact with instances through their offered pubsub API to use their functionality.
* Start, stop or restart instances at will.
* Modify the loaded configuration while running:

    * Modify config parameters from existing instances.
    * Add new instances to the config.
    * Delete instances from the config.
    * Force a reload so deleted instances are removed, added ones are created and existing ones get their updated parameters.
    * Save changes made to config to the JSON file so they persist.
    * Reload config from the JSON file so changes are lost.

### Usage examples

For example, all the instances publish their own state in the `<instance_name>.evt.mod.stat` topic. This is useful when an instance uses another instance to work properly, it subscribes to that topic and waits for that instance to be in the *started* state before starting using it (more on instance states later).

Other useful way modules use the pubsub system is when they offer an API to other modules or to other software. For example, the *socketcan* module offers the topic `<instance_name>.cmd.frame` on the pubsub to send a frame through a CAN bus.

In addition, some topics in the pubsub expect the publisher to provide a pubsub response topic to publish a response. For example, the *report* module is subscribed to topic `<instance_name>.cmd.generate`, it generates a report with all the modules status and publishes the result in the response topic the caller provided. The response can be flagged as an error too if something failed.

### Types of data

The types of data that can be published and received through the pubsub are:

* integer
* float
* string
* byte array

### Accessing the pubsub from outside

Modules use the pubsub within the obelisk binary, but two modules offer external access to allow to interact or integrate with the system.

#### Console

The `console` module offers a TCP port (by default 2323) to connect and interact with the pubsub through a terminal with software like *netcat* or *telnet*.

#### Obbus

The `obbus` module offers a TCP port (by default 2324) to connect and interact with the pubsub through a msgpack protocol that other software can implement

## Instance lifecycle

An instance can be in one of the following states:

State | Code
-- | --
Stopped | 0
Started | 1
Starting | 2
Stopping | 3
Respawning | 4

### Stopped

When an instance is instantiated it is in the `stopped` state. In this state the instance does nothing.

### Starting

An instance goes to the `starting` state in the following situations:

* When the `autostart` parameter for the instance is `true` and obelisk is starting.
* Publishing to `<instance_name>.start` or `<instance_name>.restart` on a `stopped` or `respawning` instance.
* When an instance was in `respawning` state and the respawn delay expires.

In this state the instance waits for other modules to start or does initialization tasks. 

### Started

When an instance finishes initializing and is ready to work it goes to the `started` state. In this state the instance does its normal task.

### Stopping

The `stopping` state only lasts while the instance is closing all the resources it was using, this happens in the following situations:

* If an error occurs during the normal operation of a `started` or `starting` instance it goes to `stopping` state and after closing all the resources two things can happen:
    
    * If `respawn` parameter of the instance is `true` it goes to `respawning` state.
    * Otherwise, it goes to `stopped` state.

* Publishing to `<instance_name>.stop` on a `started` or `starting` instance will cause it to go to `stopping`. After closing all the resources it goes to the `stopped` state.

* Publishing to `<instance_name>.restart` on a `started` or `starting` instance will cause it to go to `stopping`. after closing all the resources it goes to the `starting` state.

### Respawning

When an instance is in `respawning` state it waits for a respawn delay and then goes to `starting` state. The respawn delay starts being 1 second, following respawn delays will be 2, 4, 8... seconds. The maximum respawn delay will be the parameter `max_respawn_delay` of the instance. The respawn delay resets to 1 second when the instance stays in `started` state for the double of specified seconds in `max_respawn_delay`.