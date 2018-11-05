# GSR Demos

These demos work thanks to the [Net4Machines Websocket-TCP gateway](./05-ws-n4m.md) and a [GopherJS](https://github.com/gopherjs/gopherjs) wrapper of the 
[Golang obbus client](https://github.com/nayarsystems/gobbus)

## Serial 

This demo will let you connect with the serial port of the GSR, read and write data to it. (The current serial port configuration will not be modified)

<iframe width="100%" height="600" src="//jsfiddle.net/jbreva/bqso57xa/embedded/result,html,js,css/dark/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Most of the code in this demo is the Vue.js boilerplate for the buttons and text fields, but the only obbus logic is:

#### Receive serial data
```
this.bus.subscribe("serial.recv", function(msg, err) {       
          	console.log("recv:", msg)
            ...
    }
```

#### Send serial data
```
this.bus.ps("serial.cmd.write", this.sendmsginput, "")
```
