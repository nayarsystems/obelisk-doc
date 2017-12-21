# N4M Websocket-TCP Gateway

A websocket to TCP gateway for [Net4Machines](https://net4machines.com) is provided in order to access obbus on obelisk devices powered by N4M directly from the browser.

This brings all the power the pubsub provides to the browser, allowing to develop real-time applications that interact with remote devices with all the needed security.

## Starting a connection

Start a Websocket connection to:

`wss://wsgw.n4m.zone:433/?to=<device_id>.v.n4m.zone:2324&method=n4m&user=<n4m_user>&pass=<n4m_pass>`

Where:

* **<device_id>** is the ID of the obelisk device to connect to.
* **<n4m_user>** is an authorized user of N4M.
* **<n4m_pass>** is the password for the user of N4M.

## Using the connection

In that point the websocket connection works as a TCP connection to the obbus port of the device (see [obbus](./02-obbus.md)).

## An example with Go

Here's an example using the Go obbus implementation [gobbus](https://github.com/nayarsystems/gobbus) and the N4M Websocket-TCP Gateway transpiled to JavaScript to run on the web with [GopherJS](https://github.com/gopherjs/gopherjs).

This code just subscribes to the serial reception topic and prints the received frames through the browser console.

```
package main

import (
	"fmt"

	"github.com/gopherjs/gopherjs/js"
	"github.com/jaracil/wsck"
	"github.com/nayarsystems/gobbus"
)

func main() {
    conn, err := websocket.Dial("wss://wsgw.n4m.zone:443/?to=gsr.123456789.v.n4m.zone:2324&method=n4m&user=n4muser&pass=n4mpass", "http://wsorigin.example")
    if err != nil {
        println("Error connecting to websocket gateway: " + err.Error())
        return
    }
    obbusConn = gobbus.NewObbus(conn)

    rchan := make(chan *gobbus.Message, 100)
    err = obbusConn.Subscribe(rchan, "serial.recv)
    if err != nil {
        println("Error subscribing to serial recv events: " + err.Error())
        return
    }

    var msg *gobbus.Message
    var ok bool
readloop:
    for {
        select {
        case <- obbusConn.GetContext().Done():
            println("The connection was closed")
            break readloop
        case msg, ok = <-rchan:
            if ok {
                recvBuffer, err := msg.GetBuf()
                if err != nil {
                    println("Error with received message: " + err.Error())
                    break readloop
                }
                println("A message was received: " + string(recvBuffer))
            } else {
                println("Reception channel was closed")
                break readloop
            }
        }
    }
}

```