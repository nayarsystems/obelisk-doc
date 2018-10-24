# Modules

There are several modules available to Obelisk to instance, the next subsections will explain some of the most important ones.

## Socketcan

### Description

This module is in charge of configuring and managing a CAN network interface as defined in the [Linux Socketcan standard](https://www.kernel.org/doc/Documentation/networking/can.txt). The other modules receive and send CAN frames through this module.

### Configuration example

    {
        ... other instances ...
        "socketcan": {
            "module": "socketcan",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "bitrate": 200000,
            "interface_name": "can0",
            "listen_only": false,
            "loopback":	false,
            "restart_ms": 10,
            "tx_buffer_frame_num": 128
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:---  | :--- | :---
**interface_name** | Name of the CAN network interface. | `"can0"`
**bitrate** | Bitrate of the bus to which the interface is connected (the default value `-1` produces an error on the instance, so it must be configured). | `-1`
**listen_only** | Activate listening mode in which messages can be received but not sent. It also does not intervene on the bus to make the ACK of the messages. | `true`
**loopback** | Activate testing mode in which outgoing traffic is redirected as input without going through the bus. | `false`
**restart_ms** | Time in milliseconds to restart the interface automatically in case of error. | `10`
**tx_buffer_frame_num** | Size of the frame send buffer. | `128`

### Pubsub topics

Topic | Type | Description 
:---  | :--- | :---
`instance_name`.evt.frame | Byte Array | Subscribe to enable the reception of CAN frames from the CANBus.
`instance_name`.evt.error | Byte Array | Subscribe to enable the reception of CAN [error](https://github.com/torvalds/linux/blob/master/include/uapi/linux/can/error.h) frames from the CANBus.
`instance_name`.cmd.frame | Byte Array | Publish to this topic to send a CAN frame to the CANBus. An `"ok"` string is returned (or a descriptive string error).

___

## Serial
### Description

This module is in charge for configuring and managing the information through a serial port.

### Configuration example

    }
        ... other instances ...
        "serial": {
            "module": "serial",
            "log_level": "none",
            "autostart": false,
            "respawn": true,
            "max_respawn_delay": 60,
            "baudrate":	9600,
            "databits":	8,
            "hwfc":	false,
            "parity": 0,
            "read_threshold_time": 0.05,
            "stopbits":	1,
            "swfc":	false,
            "tty": "/dev/ttySERIAL"
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:---  | :---   | :---  
**tty** | Serial TTY device route. | `"/dev/ttyUSB0"`
**baudrate** | Serial baudrate. | `9600`
**databits** | Data bits of the serial (one of `8`, `7`, `6` or `5`). | `8`
**read_threshold_time** | Delay between reads to the serial in seconds, instead of reading data byte to byte. (`0` means no delay). | `0.05`
**stopbits** | Serial stop bits (one of `1` or `2`). | `1`
**parity** | Serial parity (`0` for none, `1` for odd, `2` for even). | `0`
**hwfc** | Hardware flowcontrol enabled. | `false`
**swfc** | Software flowcontrol enabled. | `false`

### Pubsub topics

Topic | Type | Description
:--- | :--- | :---
`instance_name`.recv | Byte Array | Subscribe to enable the reception of data frames from the serial bus.
`instance_name`.cmd.write | Byte Array / String | Publish to send data frames to the serial bus. The number of written bytes is returned. An error is returned if the type passed in the publish is invalid.

___

## Console

### Description

This module is responsible for opening a console that allows the user to hook up to the obbus pubsub to perform debug. It is possible to access the console through a client socket (netcat). The connection data to the console through a socket are defined in the configuration of the module.

Once connected to the socket, the available commands are the following:

Command | Value Type | Description 
:--- | :--- | :---
ps:`topic`:`value` | String  |  Publish a string message to `topic`.
pi:`topic`:`value` | Integer |  Publish a integer message to `topic`.
pf:`topic`:`value` | Float   |  Publish a float message to `topic`.
s:`topic`  | (ignored)   |  Subscribe to the messages of `topic`.
u:`topic`  | (ignored)   |  Unsubscribe from the messages of `topic`.
ua | (ignored)   | Unsubscribe from the messages of all the `topic`s.

### Configuration example

    }
        ... other instances ...
        "console": {
            "module": "console",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "max_remote_conns":	4,
            "port":	2323,
            "server_addr": "0.0.0.0",
            "timeout": 600,
            "blacklist": "^$",
            "whitelist": "^(10\\.|127\\.)"
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value
:--- | :--- | :---
**server_addr** | IP address of the server. | `"127.0.0.1"`
**port** | Server port. | `2323`
**max_remote_conns** | Maximum number of remote connections. | `4`
**timeout** | Seconds of inactivity before closing the remote connection to prevent it from being permanently open. It does not affect the local connection. A negative value disables this functionality. | `600`
**blacklist** | Regular expression to deny IP addresses of incoming connections. | `"^$"`
**whitelist** | Regular expression to accept IP addresses of incoming connections. | `"127.0.0.1"`

___

## Obbus

### Description

This module is responsible of offering a connection point with the pubsub of the system.

### Configuration example

    }
        ... other instances ...
        "obbus": {
            "module": "obbus",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "address": "0.0.0.0",
            "max_conns": 10,
            "port":	2324,
            "buffer_size": 37268,
            "timeout": 600,
            "blacklist": "^$",
            "whitelist": "^(10\\.|127\\.)"            
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**address** | IP address of the server. | `"127.0.0.1"`
**port** | Server port. | `2324`
**max_conns** | Maximum number of remote connections. | `10`
**buffer_size** | Buffer size of the obbus connections in bytes. | `37268`
**timeout** | Seconds of inactivity before closing the remote connection to prevent it from being permanently open. It does not affect the local connection. A negative value disables this functionality. | `600`
**blacklist** | Regular expression to deny IP addresses of incoming connections. | `"^$"`
**whitelist** | Regular expression to accept IP addresses of incoming connections. | `"127.0.0.1"`

___

## Slic-dahdi 

### Description

This module handles a SLIC dahdi device. Sends the appropiate tones for each state, receives and publishes the DTMFs from the SLIC.

### Configuration example

    }
        ... other instances ...
        "slic":	{
            "module": "slic-dahdi",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "busytone":	"425/200,0/200",
            "channel": 1,
            "congestiontone": "425/200,0/200,425/200,0/200,425/200,0/600",
            "dialtone":	"425",
            "dtmf_min_gain": -40,
            "dtmf_min_gap":	50,
            "dtmf_rtwist": -1,
            "dtmf_threshold": -100,
            "dtmf_twist": -1,
            "frmsize": 160,
            "lfr1":	-8,
            "lfr2":	-8,
            "ringtone":	"425/1500,0/3000"
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**channel** | Number of the dahdi device in the bus. | `1`
**frmsize** | Size of the frames in bytes. | `160`
**lfr1** | LFR1 gain (in dBs). | `-8.0`
**lfr2** | LFR2 gain (in dBs). | `-8.0`
**dialtone** | Invite to dial tone (frequency/milliseconds, ...). | `"425"`
**busytone** | Busy line tone (frequency/milliseconds, ...). | `"425/200,0/200"`
**ringtone** | Ringing tone (frequency/milliseconds, ...). | `"425/1500,0/3000"`
**congestiontone** | Busy network tone (frequency/milliseconds, ...). | `"425/200,0/200,425/200,0/200,425/200,0/600"`
**dtmf_threshold** | DTMF detection threshold. | `-100`
**dtmf_min_gain** | Minimum gain (in dBs) to detect a DTMF. | `-40.0`
**dtmf_min_gap** | Minimum gap in milliseconds between a DTMF and the previous one to detect it. | `85`
**dtmf_twist** | Gain of the high tone (in dBs). | `-1.0`
**dtmf_rtwist** | Gain of the low tone (in dBs). | `-1.0`

___

## Audiogen

### Description

This module is an auxiliary that allows to reproduce sound and publish it to other modules. It supports reading a sound file, generating a sequence of DTMFs or generating a tone.

### Configuration example

    {
        ... other instances ...   
        "audiogen":	{
            "module": "audiogen",
            "log_level": "none",
            "autostart": true,
            "respawn": true
            "max_respawn_delay": 60
	    }
        ... other instances ...
    }

___

## Gateway

### Description

This module implements the logic of a gateway between the SLIC and the modem. Allows to bypass the audio between them, to attend the call requests from the SLIC (and do redirections), to regenerate tones from the SLIC or the modem...

### Configuration example

    }
        ... other instances ...
        "gateway": {
            "module": "gateway",
            "log_level": "none",
            "autostart": false,
            "respawn": true,
            "max_respawn_delay": 60,
            "audiogen_dev":	"audiogen",
            "listen_addr": "0.0.0.0",
            "listen_port": 2325,
            "obbus_addr": "127.0.0.1",
            "obbus_port: 2324,
            "modem_data_dev": "modem",
            "modem_dev": "modem",
            "modem_dev_gain": 0,
            "modem_dtmf_q_len": 30,
            "modem_dtmf_regen": true,
            "modem_pcm_q_len": 2,
            "off_time":	15,
            "on_time": 100,
            "slic_dev": "slic",
            "slic_dev_gain": 0,
            "slic_dtmf_q_len": 30,
            "slic_dtmf_regen": true,
            "switch_polarity": false,
            "trd1": "=",
            "trd2":	"",
            "trd3":	"",
            "trd4":	"",
            "trd5":	"",
            "trd6":	"",
            "tro1":	"*",
            "tro2":	"",
            "tro3":	"",
            "tro4":	"",
            "tro5":	"",
            "tro6":	"",
            "wait_digit": 4                  
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**audiogen_dev** | Instance name of the `audiogen` module to use to send the regenerated tones from the modem to the SLIC. | `"audiogen"`
**slic_dev** | Instance name of the `slic` module to use. | `"slic"`
**modem_dev** | Instance name of the `modem` module to use. | `"modem"`
**slic_dev_gain** | Gain in dBs to apply to audio from SLIC. | `0`
**modem_dev_gain** | Gain in dBs to apply to audio from modem. | `0`
**wait_digit** | Maximum time (in seconds) to wait between digits presses of a phone from the SLIC. After it expires, the number is dialed and no more presses are expected. | `4.0`
**switch_polarity** | Generates a polarity change when an outgoing call is answered. | `false`
**tro`num`** | Translation patterns (checked in order from `num` 1 to 6) for origin number. The first pattern that matches wil be translating according to the trd`num` parameter. Examples (`"902*`, `"612345678"`, `*`). | `""`
**trd`num`** | Translation destination corresponding to the pattern tro`num`. Redirect the number to the one specified (`"="` means no redirection will be done, `""` means the call won't be done). | `""`
**modem_dtmf_regen** | Enable regeneration of the DTMFs from the modem. | `true`
**on_time** | Duration of the DTMFs regenerated from the modem. | `-8.0`
**off_time** | Duration of the silence after the DTMFs regenerated from the modem. | `0.0`
**modem_pcm_q_len** | Size of the queue for the audio (PCM) received from the modem. | `2`
**modem_dtmf_q_len** | Size of the queue for the DTMFs received from the modem. | `2`
**slic_dtmf_regen** | Enable regeneration of the DTMFs from the SLIC. | `true`
**modem_dtmf_q_len** | Size of the queue for the DTMFs received from the SLIC. | `2`
**modem_data_dev** | Instance name of the `modem` module to use in only data mode. | `""`
**listen_addr** | Address to listen for incoming connections for data calls in only data mode. | `"0.0.0.0"`
**listen_port** | Port to listen for incoming connections for data calls in only data mode. | `2325`
**obbus_addr** | Address of the obbus server to connect to | `"127.0.0.1"`
**obbus_port** | Port of the obbus server to connect to | `2324`

___

## Telealarm

### Description

This module implements the logic of a lift telealarm.

### Configuration example

    }
        ... other instances ...
        "telealarm": {
            "module": "telealarm",
            "log_level": "none",
            "autostart": false,
            "respawn": true,
            "max_respawn_delay": 60         
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**audiogen** | Instance name of the `audiogen` module to send error messages from files (disabled if left empty). | `""`
**audio_message_files** | A map containing paths for files with the voice messages. | `{}`
**modem** | Instance name of the `modem` module to use. | `""`
**audio`num`** | Instance name for each audio module (`num` from 0 to 3) corresponding to each cabin. | `""`
**modem_gain** | Gain in dBs to apply to audio from modem. | `0`
**audio_gain** | Gain in dBs to apply to audio from audio module. | `0`
**test_call_interval** | Period in seconds to do the test call | `259200`
**max_dial_attempts** | Number of retries to the list of numbers before failing. | `1`
**emerg`num`** | Emergency numbers to call (`num` from 0 to 3). | `""`
**sos`num`** | SOS numbers to call (`num` from 0 to 1). | `""`
**test`num`** | Test numbers to call (`num` from 0 to 1). | `""`
**ack_digits** | Confirmation DTMF digits for emergency calls (disabled if empty). | `""`
**ack_timeout** | Time in seconds to wait for confirmation DTMFs before stopping the call. | `10`
**ack_bypass** | Stablish audio communication while waiting the confirmation DTMFs. | `false`

___

## Modem 

### Description

This module manages a GSM modem (voice, SMS, coverage...).

### Configuration example

    {
        ... other instances ...   
    	"modem": {
            "module": "modem",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "dtmf_min_gain": -40,
            "dtmf_min_gap": 85,
            "dtmf_rtwist": -1,
            "dtmf_threshold": -100,
            "dtmf_twist": -1,
            "init_str":	"ATE0V1",
            "regen_dtmf_duration_ms": 100,
            "rx_frm_size": 800,
            "spk_vol": 7,
            "tty_cmd": "/dev/ttyCMD",
            "tty_pcm": "/dev/ttyPCM",
            "tx_frm_size": 320
	    },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**tty_cmd** | Path of the TTY device for AT commands. | `"/dev/ttyUSB0"`
**tty_pcm** | Path of the TTY device for voice. | `""`
**init_str** | Initial AT command to send to the modem. | `"ATE0V1"`
**rx_frm_size** | Reception buffer size (in bytes). | `800`
**tx_frm_size** | Transmission buffer size (in bytes). | `320`
**dtmf_threshold** | DTMF detection threshold. | `-20`
**dtmf_min_gain** | Minimum gain (in dBs) to detect a DTMF. | `-40.0`
**dtmf_min_gap** | Minimum gap in milliseconds between a DTMF and the previous one to detect it. | `85`
**dtmf_twist** | Gain of the high tone (in dBs). | `-1.0`
**dtmf_rtwist** | Gain of the low tone (in dBs). | `8.0`
**spk_vol** | Modem speaker gain (in dBs). | `7`
**data_only** | Use the modem only for data calls. | `false`
**regen_dtmf_duration_ms** | Duration in milliseconds of the regenerated tones sent with the modem. | `100`

___

## Leds

### Description

This module allows to control the LEDs of the device. In the configuration, a list of LEDs with its associated path is defined, those LEDs will have an interface in the pubsub to control them.

### Configuration example

    {
        ... other instances ...   
        "leds":	{
            "module": "leds",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "led_ath": "/sys/class/leds/ath9k-phy0",
            "led_lan": "/sys/class/leds/gsr:green:lan",
            "led_usb": "/sys/class/leds/gsr:red:usb",
            "led_wan": "/sys/class/leds/gsr:green:wan",
            "led_wlan": "/sys/class/leds/gsr:green:wlan",
            "switch_off": "lan,wan,wlan,usb"
        },
        ... other instances ...   
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**led_`name`** | Multiple configurations for multiple LEDs assigning a `name` for each one. The value is a string with de path of the led device to be controlled. | `""`
**switch_off** | A comma-separated string list with the `name`s of the LEDs to be turned off when stopping the instance. | `"lan,wan,wlan,usb"`
___

## Exec

### Description

This module is responsible of executing arbitrary commands on the target machine. To execute commands through pubsub, publish the path exec.cmd in JSON. If no timeout is specified in the JSON, the default timeout of the configuration is used; If the timeout expires before the execution of the command has finished, the execution stops. If the size of some buffer is exceeded (input, output or error of the command) an error occurs. Let's see some examples executed from the console module.

Command:

    ps:exec.cmd:{"cmd":"cat /tmp/myfile"}

Response:

    response : "{
        "exec_ok": true,
        "pid": 1234,
        "exit_ok": true,
        "output": "myfile contains some text",
        "error": "",
        "exit_code": 0
    }"

Command (with error output):

    ps:exec.cmd:{"cmd":"cat /tmp/somefile"}

Response:

    response : "{
        "exec_ok": true,
        "pid": 1235,
        "exit_ok": true,
        "output": "",
        "error": "cat: /tmp/somefile: File or directory does not exist\n",
        "exit_code": 1
    }"
 
Command (with timeout):

    { "cmd": "sleep 5", "timeout": 3 }

Response:

    response : "{
        "exec_ok": true,
        "pid": 1236,
        "exit_ok": false,
        "exit_error": "Process was killed because timeout expired after 3.000000 seconds (9 - Killed)",
        "output": "",
        "error": "",
        "exit_code": -1
    }"
 
Command (long output):

    { "cmd": "cat /tmp/loooongfile" }

Response:

    response : "{
        "exec_ok": true,
        "pid": 1237,
        "exit_ok": false,
        "exit_error": "Output exceeding max size (131072): closed pipe and truncated output",
        "output": "Some long truncated content ...",
        "error": "",
        "exit_code": -1
    }"

### Configuration example

    {
        ... other instances ...
        "exec":	{
            "module": "exec",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "max_in_size": 131072,
            "max_out_size":	131072,
            "min_out_size":	512,
            "timeout": 180
	    },
        ... other instances ...   
    }

### Specific module parameters

Parameter | Description | Default Value
:--- | :--- | :---
**timeout** | Maximum time by default to execute commands (can be modified when executing a command). | `180`
**max_in_size** | Maximum number of bytes to write as command input. | `131072`
**max_out_size** | Maximum number of bytes to read from command output. | `131072`
___

## Report

### Description

This module collects information (the current state of the system and of each instance). Once started it tries to send the report through a HTTP POST request until it succeeds. The report can be requested to the module through the pubsub too.

### Configuration example

    {
        ... other instances ...
        "report": {
            "module": "report",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "iface": "eth0",
            "max_retry_time": 3600,
            "posturl": "https://some-example-host.com/hello",
            "timeout": 10
        },
        ... other instances ...   
    }

### Specific module parameters

Parameter | Description | Default Value
:--- | :--- | :---
**timeout** | Time in seconds to wait after starting the instance to start trying to send the report. This allows for the other instances to start. | `10`
**iface** | Network interface to extract the MAC to include in the report. | `"eth0"`
**posturl** | Address to send the POST with the report. | `"http://localhost:80"`
**max_retry_timeout** | Maximum time in seconds to wait between tries to send the report (it increases linearly at a reason of 10 seconds until this maximum). | `3600`

___

## Wdinet

### Description

This module is responsible of tracking the network interfaces specified in the configuration and modifying the metrics of its default routes to prioritize the Internet access for those who have real access to Internet.

Each interface is checked if it has a default route, if it has one it tries to execute a ping towards each of the hosts in ping_list, if any of the pings works it is determined that there is internet access by said interface and the metric of its default route moves to the 0-19 interval, making it a higher priority than those that do not have ping (metrics 20-39).

The states of each interface will be published as JSON in the topic wdinet.status.`interface_name`.

If after a time determined by `panic_timeout` no ping has been achieved by any interface, the `panic_cmd` command will be executed.

### Configuration example

    {
        ... other instances ...
        "wdinet": {
            "module": "wdinet",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "awk_path":	"awk",
            "clean_untracked": true,
            "head_path": "head",
            "ifaces": "eth0,eth1,wlan0,br-lan,ppp0",
            "ip_path": "ip",
            "off_period": 30,
            "on_period": 300,
            "panic_cmd": "reboot",
            "panic_timeout": 900,
            "ping_list": "8.8.8.8,4.4.4.4",
            "ping_path": "ping",
            "ping_tout": 4,
            "route_path": "route"
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value
:--- | :--- | :---
**ifaces** | Comma-separated string list of interfaces to track. | `"eth0,eth1,wlan0,br-lan,ppp0"`
**clean_untracked** | Prioritize the tracked interfaces to the others of the system (moving the others to higher metrics). | `false`
**ping_list** | Comma-separated string list of hosts to ping. | `"8.8.8.8,8.8.4.4"`
**off_period** | Interval in seconds to check the interfaces when no ping has succeeded. | `10.0`
**on_period** | Interval in seconds to check the interfaces when a ping has succeeded. | `60.0`
**ping_path** | Command to execute for the ping utility. | `"ping"`
**ping_tout** | Timeout in seconds for the pings. | `4.0`
**ip_path** | Command to execute for the ip utility. | `"ip"`
**route_path** | Command to execute for the route utility. | `"route"`
**awk_path** | Command to execute for the awk utility. | `"awk"`
**head_path** | Command to execute for the head utility. | `"head"`
**panic_tout** | Time in seconds without ping to execute the panic command. | `900`
**panic_cmd** | Command to execute when the panic timeout expires (none if left empty). | `"reboot"`

___

## 3g

### Description

This module is responsible for making the connection to the Internet through the 3G modem. Make the conversation with the modem to configure it, test APNs through the world list or allow it to be configured by hand; run the PPPD daemon with the correct configuration and check that there is internet access through 3G with the `wdinet` module. If you do not get internet after a certain time, a panic command is executed.

### Configuration example

    {
        ... other instances ...
        "3g":	{
            "module": "3g",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "apn_file":	"/etc/apns.json",
            "force_apn": "",
            "force_apn_pass": "",
            "force_apn_user": "",
            "metric": 30,
            "modem_init": "ATE0V1",
            "panic_cmd": "/usr/bin/panic_cmd.sh",
            "panic_timeout": 600,
            "pppd_params": "mtu 1450",
            "pppd_path": "pppd",
            "pppd_wait_iface": 25,
            "timeout": 5,
            "tty_hwc": false,
            "tty_path": "/dev/ttyPPP",
            "unit":	0
	    },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value
:--- | :--- | :---
**tty_path** | Path of the modem TTY device. | `"/dev/ttyUSB0"`
**tty_hwc** | Enable TTY hardware flowcontrol. | `false`
**timeout** | Timeout in seconds to wait for AT command responses. | `5.0`
**modem_init** | Initial AT command to send to the modem. | `"ATE0V1"`
**pppd_path** | Command to execute the pppd utility. | `"pppd"`
**pppd_params** | Extra parameters to pass to pppd. | `"mtu 1450"`
**pppd_wait_iface** | Time in seconds to wait for the interface to be ready after launching pppd. | `25`
**unit** | Number of the desired interface to pass to pppd, used to name the interface (pppX). | `0`
**metric** | Desired metric of the interface to pass to pppd (see `wdinet` module). | `30`
**apn_file** | Path of the JSON file to use as list of APNs. | `"/etc/apns.json"`
**force_apn** | Force the specified APN (if empty the file with the list of APNs is used). | `""`
**force_apn_user** | User to use with the forced APN. | `""`
**force_apn_pass** | Password to use with the forced APN. | `""`
**panic_timeout** | Time in seconds without internet through 3G to execute the panic command. | `600`
**panic_cmd** | Command to execute when the panic timeout expires (none if left empty). | `""`

___

## Sms 

### Description

This module acts as a bridge between the received SMSs (through a `modem` instance) and the pubsub.

The format of the SMS iss:

    SMS = <password>,<command>,<command>...
    S:(<param>|<$param_alias>):<value> = sys.cmd.cfg.set.param
    value == "t" > true
    value == "f" > false
    S! > force to set the param although it doesn't exists or type differs
    G:(<param>|<$param_alias>) = sys.cmd.cfg.get
    MS:(<param>|<$param_alias>)
    MS = Module Restart
    MK = Module Stop 
    MR = Module Restart
    ML = Module Reload
    I:<param> = Add <param> to response (??)
    R = Respond
    NR = No respond
    W = Save cfg

### Configuration example

    {
        ... other instances ...
        "sms":	{
            "module": "sms",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "$ackbpass": "telealarm.ack_bypass",
            "$ackdtmf":	"telealarm.ack_digits",
            "$acktout":	"telealarm.ack_timeout",
            "$apn":	"3g.force_apn",
            "$apnp": "3g.force_apn_pass",
            "$apnu": "3g.force_apn_user",
            "$ci": "telealarm.test_call_interval",
            "$e0": "telealarm.emerg0",
            "$e1": "telealarm.emerg1",
            "$e2": "telealarm.emerg2",
            "$e3": "telealarm.emerg3",
            "$mda":	"telealarm.max_dial_attempts",
            "$saa":	"alert_acqcard.sms_to",
            "$sab":	"alert_battery.sms_to",
            "$sat":	"alert_telealarm.sms_to",
            "$sos0": "telealarm.sos0",
            "$sos1": "telealarm.sos1",
            "$swpl": "gateway.switch_polarity",
            "$test0": "telealarm.test0",
            "$test1": "telealarm.test1",
            "$trd1": "gateway.trd1",
            "$trd2": "gateway.trd2",
            "$trd3": "gateway.trd3",
            "$trd4": "gateway.trd4",
            "$trd5": "gateway.trd5",
            "$trd6": "gateway.trd6",
            "$tro1": "gateway.tro1",
            "$tro2": "gateway.tro2",
            "$tro3": "gateway.tro3",
            "$tro4": "gateway.tro4",
            "$tro5": "gateway.tro5",
            "$tro6": "gateway.tro6",
            "modem_dev": "modem",
            "password":	"12345",
            "respond": true
	    },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**modem_dev** | Modem module instance name from which to receive SMSs. | `"modem"`
**password** | Password the SMS must contain to be valid. | `"22sms22"`
**respond** | Respond to all the SMSs (included those that don't requested a response). | `true`
**report_parameters** | A map to specify the data to be returned by a SMS report. The keys correspond to a section (an instance name or `main`) in the report generated by the `report` module and the values correpond to the parameter of that section to be included in the report. | `{"main": "id", "console": "remote_conns"}`
**$`param_alias`** | One alias for each parameter of the config to use with the SMSs, instead of writing the instance and the parameter, the alias can be used to save characters on the SMS. | 

___

## Battery

### Description

This module is responsible of periodically checking the battery status, publishing the read values of the battery in the pubsub and generating alerts depending on the threshold values defined for each variable. Works with a BQ34Z100 module.

### Configuration example

    {
        ... other instances ...
       	"battery":	{
            "module": "battery",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "i2cbus": "/dev/i2c-0",
            "reload_period": 60,
            "vars":	[{
                    "alert": true,
                    "alert_low": 560,
                    "alert_safe": 590,
                    "name":	"v",
                    "thold": 2
                },
                {
                    "alert": false,
                    "name":	"i",
                    "thold": 2
                },
                {
                    "alert": true,
                    "alert_max": 50,
                    "name":	"t",
                    "thold": 2
                },
                {
                    "alert": false,
                    "name":	"soc",
                    "thold": 2
                }]
	    }, 
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value
:--- | :--- | :---
**i2cbus** | I2C device path. | `"/dev/i2c-0"`
**reload_period** | Interval in seconds to read and publish battery data. | `60`
**vars** | A list for each battery variable defining the thresholds and alert values. | `[]`

#### vars parameters

Parameter | Description
:--- | :---
**name** | Name of the variable (`"v"` for voltage, `"i"` for intensity, `"t"` for temperature and `"soc"` for state of charge).
**thold** | Threshold to publish again the read data (from the last published value).
**alert** | Wheter to activate the alerts for the variable (`true`) or not (`false`).
**alert_level** | *(only for `v`)* Limit value to generate the alert state.
**alert_safe** | *(only for `v`)* Limit value to generate the alert recovery state.
**alert_max** | *(only for `t`)* Limit value to generate the alert state.

___

## Watchdog

### Description

This module is responsible for opening and maintaining the system watchdog according to the [Linux documentation](https://www.kernel.org/doc/Documentation/watchdog/watchdog-api.txt).

### Configuration example

    {
        ... other instances ...
        "watchdog":	{
            "module": "watchdog",
            "log_level": "none",
            "autostart": true,
            "respawn": true
            "max_respawn_delay": 60,
            "auto_close": true,
            "auto_open": true,
            "interval":	1,
            "path":	"/dev/watchdog"
	    },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**path** | Path to watchdog device. | `"/dev/watchdog"`
**interval** | Time in seconds to wait between writes to watchdog. | `1`
**auto_open** | Open the watchdog automatically when starting the instance. | `true`
**auto_close** | Close the watchdog automatically when stopping the instance. | `true`

___

## Vpn4m

### Description

This module is responsible of connecting to the net4machines VPN.

### Configuration example

    {
        ... other instances ...
        "n4m": {
            "module": "vpn4m",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "fail_cmd_error": false,
            "host":	"eu1.z.n4m.zone",
            "if_name": "n4m%d",
            "ip_cmd": "/usr/sbin/ip",
            "ipt_cmd": "/usr/sbin/iptables",
            "port":	7744,
            "user":	"user"
            "password":	"password"
	    },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**host** | Address of the VPN server to connect. | `"eu1.z.n4m.zone"`
**port** | Port of the VPN server to connect. | `7744`
**user** | User to authenticate. | `"user"`
**password** | Password to authenticate. | `"password"`
**if_name** | Name of the created interface. | `"n4m%d"`
**fail_cmd_error** | Treat as an error the execution of commands with a return value different to zero. | `false`
**restart_on_default_route** | Restart the instance when a change on the default route is found through the `wdinet` instance. | `true`
**ip_cmd** | Command to execute the ip utility. | `"/usr/sbin/ip"`
**iptables_cmd** | Command to execute the iptables utility. | `"/usr/sbin/iptables"`

___

## Nexus

### Description

This module opens a [nexus](https://github.com/jaracil/nexus) connection.

___

## Evdev

### Description

This module captures the input events of an evdev device and publishes them on the obelisk pubsub. This, for example, allows to react to button pushes.

### Configuration example

    {
        ... other instances ...
        "evdev": {
            "module": "evdev",
            "log_level": "none",
            "autostart": true,
            "respawn": true
            "max_respawn_delay": 60,
            "devpath": "/dev/input/event0"
	    },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**devpath** | Path to the device to be watched by the instance. | `"/dev/input/button"`

___

## Alert

### Description

This module is reponsible of subscribing to the alerts that you wish to notify, and notifies those alerts by the configured channel (SMS, POST, etc).

The module currently allows to configure a maximum of two channels to notify the alert: HTTP POST and SMS. The notification addresses are defined by configuring the parameters `sms_to` and `url_post`. If both methods are configured, HTTP POST is given priority because it is the most economical method.

The timeout parameter is used when all the methods configured to notify the alert fail, initiating an exponential timer using `timeout` as the base time. In addition, a `max_ttl` time is defined to discard the alert as soon as that time is reached without being notified.

### Configuration example

    {
        ... other instances ...
        "alert_battery": {
            "module": "alert",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "modem_dev": "modem",
            "alert_path": "battery",
            "max_ttl": 86400,
            "sms_to": "",
            "timeout": 3600,
            "url_post":	"https://some-example-host.com/alert/battery"
	    },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**alert_path** | Path of the pubsub to subscribe for receiving alerts (`sys.alert.<alert_path>`). | `""`
**sms_to** | Phone number to send the SMS to. Not send by SMS if empty. | `""`
**modem_dev** | Instance name of the `modem` module to send the SMSs. | `"modem"`
**url_post** | Address to send the alert through POST. Not sent by POST if empty. | `""`
**timeout** | Time in seconds to wait between tries to send the alert. It increments exponentially. | `3600`
**max_ttl** | Maximum life time in seconds for the alert, when this time expires and the alert has not been notified, it's deleted. | `86400`

___

## Button

### Description

This module implements the logic of a button. The controlled key is identified by the type and by the code issued by the **mod_evdev**. If the monitored key is kept pressed for at least `keypress_min_secs` seconds and no more than `keypress_max_secs`, the list of actions defined in the `actions` parameter tag is executed. 

This `actions` parameter is an array of json objects, each with three keys: topic, type and payload. The topic identifies the topic that is published in the pubsub, the type defines the type of what is published, and the payload the data field that is published. With this list of `actions` we can make the button inject in the pubsub the configured commands.

### Configuration example

    {
        ... other instances ...
        "button_config": {
            "module": "button",
            "log_level": "none",
            "autostart": true,
            "respawn": true,
            "max_respawn_delay": 60,
            "actions": [{
                "payload": "{\"cmd\":\"/usr/bin/btn_config.sh\", \"timeout\":120}",
                "topic": "exec.cmd",
                "type": "s"
            }],
            "code":	529,
            "keypress_max_secs": 6,
            "keypress_min_secs": 2,
            "type":	1
        },
        "button_factory": {
            "module": "button",
            "autostart": true,
            "log_level": "none",
            "respawn": true,
            "max_respawn_delay": 60,
            "actions": [{
                "payload": "{\"cmd\":\"/usr/bin/btn_reset_factory.sh\", \"timeout\":1200}",
                "topic": "exec.cmd",
                "type":	"s"
            }],
            "code":	529,
            "keypress_max_secs": 60,
            "keypress_min_secs": 15,
            "type":	1
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**type** | Type of event (/usr/include/linux/input-event-codes.h). | `255`
**code** | Key code (/usr/include/linux/input-event-codes.h). | `255`
**keypress_min_secs** | Minimum number of seconds with the button pressed to execute the actions. | `0`
**keypress_min_secs** | Maximum number of seconds with the button pressed to execute the actions. | `60`
**actions** | JSON string with a list of the actions as defined in the description. | `"[]"`

___

## Gsrleds

### Description

This module is reponsible of subscribing to GSR events that require turning LEDs on or off on the GSR, and manage the LEDs accordingly.

### Configuration example

    {
        ... other instances ...
        "gsrleds": {
            "module": "gsrleds",
            "log_level": "none",
            "autostart": true,
            "respawn": true
            "max_respawn_delay": 60,
            "battery_led": "wan",
            "config_led": "lan",
            "modem_green_led": "wlan",
            "modem_red_led": "usb",
            "modem_dev": "modem",
            "gateway_dev": "gateway",
            "telealarm_dev": "telealarm"
	    },
        ... other instances ...
    }

___

## Events

### Description

This module is reponsible of aggregate pubsub messages on a path, and POST them to a server. 

The posted events have the following format:
    
    POST /events/add - GSR
    Content-Type: application/json, Content-Length: 153, Accept-Encoding: gzip
    {  
        "max17048.evt.v":{  
            "info":"These are voltage values",
            "timestamp":1540307066461, // Unix timestamp in milliseconds
            "delta":[0,10040, 20133, 30179, 40219],
            "values":[8.2675, 8.2675, 8.2675, 8.2675, 8.2675]
        }
    }

### Configuration example

    {
        ... other instances ...
        "events": {
                "module": "events",
                "topic": "test.*",
                "parse_json": true,
                "filter_json": "a,b,c,d",
                "post_url": "http://localhost:8000/",
                "send_max_elements": 10,
                "send_period_s": 300
        },
        ... other instances ...
    }

### Specific module parameters

Parameter | Description | Default Value 
:--- | :--- | :---
**topic** | Pubsub topic to subscribe to | `""`
**parse_json** | Flag that the messages posted to the subscribed path are json encoded strings and we will want to parse them | `false`
**filter_json** | If **parse_json** is true, extract only this comma-separated list of keys | `""`
**post_url** | URL to POST the events to | `""`
**info_str** | Static string that will be attached to the POSTed object  | `""`
**send_max_elements** | Maximum number of aggregated elements before posting | `1024`
**send_period_s** | Maximum seconds to wait before posting | `300`

### Pubsub topics

Topic | Type | Description 
:---  | :--- | :---
`instance_name`.cmd.flush | * | Send the aggregated events now, without waiting for the time or elemetns limit
