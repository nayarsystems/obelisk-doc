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

| Parameter               | Description                                                                                                                                    | Default Value |
| :---------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| **interface_name**      | Name of the CAN network interface.                                                                                                             | `"can0"`      |
| **bitrate**             | Bitrate of the bus to which the interface is connected (the default value `-1` produces an error on the instance, so it must be configured).   | `-1`          |
| **listen_only**         | Activate listening mode in which messages can be received but not sent. It also does not intervene on the bus to make the ACK of the messages. | `true`        |
| **loopback**            | Activate testing mode in which outgoing traffic is redirected as input without going through the bus.                                          | `false`       |
| **restart_ms**          | Time in milliseconds to restart the interface automatically in case of error.                                                                  | `10`          |
| **tx_buffer_frame_num** | Size of the frame send buffer.                                                                                                                 | `128`         |

### Pubsub topics

| Topic                     | Type       | Description                                                                                                                                            | Flags                                |
| :------------------------ | :--------- | :----------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------- |
| *evt*                     |            |                                                                                                                                                        |
| `instance_name`.evt.frame | Byte Array | Subscribe to enable the reception of CAN frames from the CANBus.                                                                                       | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.error | Byte Array | Subscribe to enable the reception of CAN [error](https://github.com/torvalds/linux/blob/master/include/uapi/linux/can/error.h) frames from the CANBus. |
| *cmd*                     |            |                                                                                                                                                        |
| `instance_name`.cmd.frame | Byte Array | Publish to this topic to send a CAN frame to the CANBus. An `"ok"` string is returned (or a descriptive string error).                                 |

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

| Parameter               | Description                                                                                               | Default Value    |
| :---------------------- | :-------------------------------------------------------------------------------------------------------- | :--------------- |
| **tty**                 | Serial TTY device route.                                                                                  | `"/dev/ttyUSB0"` |
| **baudrate**            | Serial baudrate.                                                                                          | `9600`           |
| **databits**            | Data bits of the serial (one of `8`, `7`, `6` or `5`).                                                    | `8`              |
| **read_threshold_time** | Delay between reads to the serial in seconds, instead of reading data byte to byte. (`0` means no delay). | `0.05`           |
| **stopbits**            | Serial stop bits (one of `1` or `2`).                                                                     | `1`              |
| **parity**              | Serial parity (`0` for none, `1` for odd, `2` for even).                                                  | `0`              |
| **hwfc**                | Hardware flowcontrol enabled.                                                                             | `false`          |
| **swfc**                | Software flowcontrol enabled.                                                                             | `false`          |

### Pubsub topics

| Topic                           | Type                | Description                                                                                                                                                | Flags                                |
| :------------------------------ | :------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------- |
| *recv*                          |                     |                                                                                                                                                            |
| `instance_name`.recv            | Byte Array          | Subscribe to enable the reception of data frames from the serial bus.                                                                                      |
| *evt*                           |                     |
| `instance_name`.evt.recv        | Byte Array          | Subscribe to enable the reception of data frames from the serial bus.                                                                                      | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.rts         | Integer             | Publish to set rts.                                                                                                                                        |
| `instance_name`.evt.cts         | Integer             | Publish to set cts.                                                                                                                                        |
| `instance_name`.evt.dtr         | Integer             | Publish to set dtr.                                                                                                                                        |
| `instance_name`.evt.dsr         | Integer             | Publish to set dsr.                                                                                                                                        |
| `instance_name`.evt.dcd         | Integer             | Publish to set dcd.                                                                                                                                        |
| `instance_name`.evt.ri          | Integer             | Publish to set ri.                                                                                                                                         |
| *cmd*                           |                     |
| `instance_name`.cmd.write       | Byte Array / String | Publish to send data frames to the serial bus. The number of written bytes is returned. An error is returned if the type passed in the publish is invalid. |
| `instance_name`.cmd.setbaudrate | Integer             | Publish to set baudarate.                                                                                                                                  |
| `instance_name`.cmd.setparity   | Integer             | Publish to set parity.                                                                                                                                     |
| `instance_name`.cmd.getrts      | Integer             | Publish to get rts.                                                                                                                                        |
| `instance_name`.cmd.getcts      | Integer             | Publish to get cts.                                                                                                                                        |
| `instance_name`.cmd.getdtr      | Integer             | Publish to get dtr.                                                                                                                                        |
| `instance_name`.cmd.getdsr      | Integer             | Publish to get dsr.                                                                                                                                        |
| `instance_name`.cmd.getdcd      | Integer             | Publish to get dcd.                                                                                                                                        |
| `instance_name`.cmd.getri       | Integer             | Publish to get ri.                                                                                                                                         |

___

## Console

### Description

This module is responsible for opening a console that allows the user to hook up to the obbus pubsub to perform debug. It is possible to access the console through a client socket (netcat). The connection data to the console through a socket are defined in the configuration of the module.

Once connected to the socket, the available commands are the following:

| Command            | Value Type | Description                                       |
| :----------------- | :--------- | :------------------------------------------------ |
| ps:`topic`:`value` | String     | Publish a string message to `topic`.              |
| pi:`topic`:`value` | Integer    | Publish a integer message to `topic`.             |
| pf:`topic`:`value` | Float      | Publish a float message to `topic`.               |
| s:`topic`          | (ignored)  | Subscribe to the messages of `topic`.             |
| u:`topic`          | (ignored)  | Unsubscribe from the messages of `topic`.         |
| ua                 | (ignored)  | Unsubscribe from the messages of all the `topic`. |

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

| Parameter            | Description                                                                                                                                                                                  | Default Value |
| :------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| **server_addr**      | IP address of the server.                                                                                                                                                                    | `"127.0.0.1"` |
| **port**             | Server port.                                                                                                                                                                                 | `2323`        |
| **max_remote_conns** | Maximum number of remote connections.                                                                                                                                                        | `4`           |
| **timeout**          | Seconds of inactivity before closing the remote connection to prevent it from being permanently open. It does not affect the local connection. A negative value disables this functionality. | `600`         |
| **blacklist**        | Regular expression to deny IP addresses of incoming connections.                                                                                                                             | `"^$"`        |
| **whitelist**        | Regular expression to accept IP addresses of incoming connections.                                                                                                                           | `"127.0.0.1"` |

### Pubsub topics

| Topic        | Type                       | Description                                                                                                                                                                                                                                                                                                                                     |
| :----------- | :------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `topic`      | String/Integer/Double/Bool | The topic is received and depending on the first element, one action or another is performed. ps -> Post a string, pi-> Post an integer, pf -> Post a float, s-> subscribe to a topic, u-> unsubscribe, ua-> unsubscribe from all topics. `Ex:  s:max17048.*, pi:pdahl.cmd.send:2, ps:modem.evt.sms:{"from":"654321234", "body":"12345,C:rp"} ` |
| *sys*        |                            |                                                                                                                                                                                                                                                                                                                                                 |
| sys.cmd.exit | String                     | Publish cmd exit                                                                                                                                                                                                                                                                                                                                |

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

| Parameter       | Description                                                                                                                                                                                  | Default Value |
| :-------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| **address**     | IP address of the server.                                                                                                                                                                    | `"127.0.0.1"` |
| **port**        | Server port.                                                                                                                                                                                 | `2324`        |
| **max_conns**   | Maximum number of remote connections.                                                                                                                                                        | `10`          |
| **buffer_size** | Buffer size of the obbus connections in bytes.                                                                                                                                               | `37268`       |
| **timeout**     | Seconds of inactivity before closing the remote connection to prevent it from being permanently open. It does not affect the local connection. A negative value disables this functionality. | `600`         |
| **blacklist**   | Regular expression to deny IP addresses of incoming connections.                                                                                                                             | `"^$"`        |
| **whitelist**   | Regular expression to accept IP addresses of incoming connections.                                                                                                                           | `"127.0.0.1"` |

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

| Parameter          | Description                                                                   | Default Value                                 |
| :----------------- | :---------------------------------------------------------------------------- | :-------------------------------------------- |
| **channel**        | Number of the dahdi device in the bus.                                        | `1`                                           |
| **frmsize**        | Size of the frames in bytes.                                                  | `160`                                         |
| **lfr1**           | LFR1 gain (in dBs).                                                           | `-8.0`                                        |
| **lfr2**           | LFR2 gain (in dBs).                                                           | `-8.0`                                        |
| **dialtone**       | Invite to dial tone (frequency/milliseconds, ...).                            | `"425"`                                       |
| **busytone**       | Busy line tone (frequency/milliseconds, ...).                                 | `"425/200,0/200"`                             |
| **ringtone**       | Ringing tone (frequency/milliseconds, ...).                                   | `"425/1500,0/3000"`                           |
| **congestiontone** | Busy network tone (frequency/milliseconds, ...).                              | `"425/200,0/200,425/200,0/200,425/200,0/600"` |
| **dtmf_threshold** | DTMF detection threshold.                                                     | `-100`                                        |
| **dtmf_min_gain**  | Minimum gain (in dBs) to detect a DTMF.                                       | `-40.0`                                       |
| **dtmf_min_gap**   | Minimum gap in milliseconds between a DTMF and the previous one to detect it. | `85`                                          |
| **dtmf_twist**     | Gain of the high tone (in dBs).                                               | `-1.0`                                        |
| **dtmf_rtwist**    | Gain of the low tone (in dBs).                                                | `-1.0`                                        |

### Pubsub topics

| Topic                            | Type    | Description                                                         | Flags                                |
| :------------------------------- | :------ | :------------------------------------------------------------------ | :----------------------------------- |
| *evt*                            |         |                                                                     |
| `instance_name`.evt.onhook       | Integer | Publish to set value to ringer and post to cmd.ringer.              |
| `instance_name`.evt.pcm          | Integer | Publish to  value to generate dtfm_rx.                              | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.pulsestart   | String  | Publish to activate the ev_slic_pulsestart of the gw state machine. |
| `instance_name`.evt.pulsestop    | Integer | Publish to disable  the ev_slic_pulsestart of the gw state machine. |
| `instance_name`.evt.needpcm      | Integer | Publish to size buffer pcm.                                         | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.dtmf         | String  | Publish to dtmf.                                                    |
| `instance_name`.evt.dtmf_discard | String  | Publish to dtmf discard.                                            |
| `instance_name`.evt.dtmf_end     | String  | Publish to end of dtmf                                              |
| `instance_name`.evt.pcm_echocan  | Buffer  | Publish to buffer                                                   | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |

*cmd* | |
`instance_name`.cmd.pcmsend | Byte Array / String | Publish to buffer to the CAN.
`instance_name`.cmd.playtone | String | The selected tone is played. Options: *dial, busy, ring, congestion, none*.
`instance_name`.cmd.echobuf | Integer | The value is stored in *ctx->echobuf*.
`instance_name`.cmd.echoflags | Integer | The value is stored in *ctx->echoflags* and modification to flags.
`instance_name`.cmd.polarity | Integer | Publish set polarity
`instance_name`.cmd.dtmfsend | String | Publish to Json with information of *type, digits, on_time, off_time, level, twist* and publish in *cmd.play*
`instance_name`.cmd.ringer | String | Publish set ctx->ringer and activate o disable
`instance_name`.cmd.play | String | Publish to Json with information of *type, digits, on_time, off_time, level, twist*.
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

### Pubsub topics

| Topic                                 | Type    | Description                                                            |
| :------------------------------------ | :------ | :--------------------------------------------------------------------- |
| *evt*                                 |         |
| `instance_name`.evt.started.`topic`   | Integer | Publish activate to topic                                              |
| `instance_name`.evt.stopped.`topic`   | Integer | Publish disable to topic                                               |
| `instance_name`.evt.needpcm           | Integer | Publish verify the configuration and send a publication of evt.started |
| *cmd*                                 |         |
| `instance_name`.cmd.stop.`topic`      |         | Publish close audiogen                                                 |
| `instance_name`.cmd.play.*            |         | Publish activate audiogen and post *cmd.isplaying*                     |
| `instance_name`.cmd.isplaying.`topic` | String  | Pubblish to topic and return 1.                                        |


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

| Parameter            | Description                                                                                                                                                                                                 | Default Value |
| :------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| **audiogen_dev**     | Instance name of the `audiogen` module to use to send the regenerated tones from the modem to the SLIC.                                                                                                     | `"audiogen"`  |
| **slic_dev**         | Instance name of the `slic` module to use.                                                                                                                                                                  | `"slic"`      |
| **modem_dev**        | Instance name of the `modem` module to use.                                                                                                                                                                 | `"modem"`     |
| **slic_dev_gain**    | Gain in dBs to apply to audio from SLIC.                                                                                                                                                                    | `0`           |
| **modem_dev_gain**   | Gain in dBs to apply to audio from modem.                                                                                                                                                                   | `0`           |
| **wait_digit**       | Maximum time (in seconds) to wait between digits presses of a phone from the SLIC. After it expires, the number is dialed and no more presses are expected.                                                 | `4.0`         |
| **switch_polarity**  | Generates a polarity change when an outgoing call is answered.                                                                                                                                              | `false`       |
| **tro`num`**         | Translation patterns (checked in order from `num` 1 to 6) for origin number. The first pattern that matches wil be translating according to the trd`num` parameter. Examples (`"902*`, `"612345678"`, `*`). | `""`          |
| **trd`num`**         | Translation destination corresponding to the pattern tro`num`. Redirect the number to the one specified (`"="` means no redirection will be done, `""` means the call won't be done).                       | `""`          |
| **modem_dtmf_regen** | Enable regeneration of the DTMFs from the modem.                                                                                                                                                            | `true`        |
| **on_time**          | Duration of the DTMFs regenerated from the modem.                                                                                                                                                           | `-8.0`        |
| **off_time**         | Duration of the silence after the DTMFs regenerated from the modem.                                                                                                                                         | `0.0`         |
| **modem_pcm_q_len**  | Size of the queue for the audio (PCM) received from the modem.                                                                                                                                              | `2`           |
| **modem_dtmf_q_len** | Size of the queue for the DTMFs received from the modem.                                                                                                                                                    | `2`           |
| **slic_dtmf_regen**  | Enable regeneration of the DTMFs from the SLIC.                                                                                                                                                             | `true`        |
| **modem_dtmf_q_len** | Size of the queue for the DTMFs received from the SLIC.                                                                                                                                                     | `2`           |
| **modem_data_dev**   | Instance name of the `modem` module to use in only data mode.                                                                                                                                               | `""`          |
| **listen_addr**      | Address to listen for incoming connections for data calls in only data mode.                                                                                                                                | `"0.0.0.0"`   |
| **listen_port**      | Port to listen for incoming connections for data calls in only data mode.                                                                                                                                   | `2325`        |

___
### Pubsub topics

| Topic                                     | Type                | Description                                                                     | Flags                               |
| :---------------------------------------- | :------------------ | :------------------------------------------------------------------------------ | :---------------------------------- |
| *evt*                                     |                     |                                                                                 |
| `instance_name`.evt.timeout               | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name`.evt.deactivate_timeout    | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name`.evt.modem_dtmf_timeout    | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name`.evt.slic_dtmf_timeout     | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name`.evt.dial_result           | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name`.evt.stat                  | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_dialer`.evt.bstat          | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_dialer_data`.evt.bstat     | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_dialer`.evt.pcm            | Integer             | Publish activate to topic                                                       |
| `instance_name_dialer`.evt.dtfm           | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_dialer`.evt.dtfm_end       | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name`.evt.slic_dtmfsent         | Integer             |
| `instance_name_tas`.evt.end               | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name`.evt.set_tas_params_result | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_slic`.evt.onhook           | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_slic`.evt.pulsestart       | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_slic`.evt.pulse            | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_slic`.evt.pulsestop        | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_slic`.evt.ready            | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_slic`.evt.dtfm             | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_slic`.evt.dtfm_end         | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name`.evt.modem_dtmfsent        | Integer             | Depending on the TYPE the state machine is modified.                            |
| `instance_name_slic`.evt.pcm              | Integer             | Publish activate to topic                                                       |
| *cmd*                                     |                     |                                                                                 |
| `instance_name`.cmd.deactivate            | Integer             | Publish disactivate to gateway                                                  |
| `instance_name`.cmd.activate              | Integer             | Publish activate to gateway                                                     |
| `instance_name`.cmd.set_state             | String              | Publish set to state (*online, ringing, dialout, busy, congestion, idle*)       |
| `instance_name_dialer_data`.cmd.datasend  | Byte Array / String | Publish sends the contents of the buffer.                                       |
| **`instance_name_slic`.cmd.playtone       | String              | Publish sends to tone is played. Options: *dial, busy, ring, congestion, none*. |
| **`instance_name_slic`.cmd.ringer         | Integer             | Publish activate to topic                                                       |
| `instance_name_dialer_data`.cmd.hangup    | Integer             | Publish send 1 to hang up.                                                      |
| `instance_name_dialer`.cmd.hangup         | Integer             | Publish send 1 to hang up.                                                      |
| `instance_name_dialer`.cmd.answer         | Integer             | The answer is sent to the dialer and it is sent to the corresponding module.    |
| **`instance_name_slic`.cmd.polarity       | Integer             | Publish set polarity                                                            |
| `instance_name_tas`.cmd.set_params        | String              | Publish the parameters of the tas                                               |
| `instance_name_dialer`.cmd.dtmfsend       | String              | Publish send to dtmf.                                                           |
| `instance_name_slic`.cmd.dtfmsend         | String              | Publish send to dtmf.                                                           |
| `instance_name_dialer`.cmd.pcmsend        | Buffer              | Publish send to buffer.                                                         | MSG_FL_INSTANT/ MSG_FL_NONRECURSIVE |


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
            "max_respawn_delay": 60,
            "interphone_enabled": true        
        },
        ... other instances ...
    }

### Specific module parameters

| Parameter               | Description                                                                                        | Default Value |
| :---------------------- | :------------------------------------------------------------------------------------------------- | :------------ |
| **audiogen**            | Instance name of the `audiogen` module to send error messages from files (disabled if left empty). | `""`          |
| **audio_message_files** | A map containing paths for files with the voice messages.                                          | `{}`          |
| **modem**               | Instance name of the `modem` module to use.                                                        | `""`          |
| **audio`num`**          | Instance name for each audio module (`num` from 0 to 3) corresponding to each cabin.               | `""`          |
| **modem_gain**          | Gain in dBs to apply to audio from modem.                                                          | `0`           |
| **audio_gain**          | Gain in dBs to apply to audio from audio module.                                                   | `0`           |
| **test_call_interval**  | Period in seconds to do the test call                                                              | `259200`      |
| **max_dial_attempts**   | Number of retries to the list of numbers before failing.                                           | `1`           |
| **emerg`num`**          | Emergency numbers to call (`num` from 0 to 3).                                                     | `""`          |
| **sos`num`**            | SOS numbers to call (`num` from 0 to 1).                                                           | `""`          |
| **test`num`**           | Test numbers to call (`num` from 0 to 1).                                                          | `""`          |
| **ack_digits**          | Confirmation DTMF digits for emergency calls (disabled if empty).                                  | `""`          |
| **ack_timeout**         | Time in seconds to wait for confirmation DTMFs before stopping the call.                           | `10`          |
| **ack_bypass**          | Stablish audio communication while waiting the confirmation DTMFs.                                 | `false`       |
| **interphone_enabled**  | enabled or disable interphone                                                                      | `true`        |

### Pubsub topics

| Topic                                                      | Type    | Description                                                                                              | Flags                                |
| :--------------------------------------------------------- | :------ | :------------------------------------------------------------------------------------------------------- | :----------------------------------- |
| *evt*                                                      |         |                                                                                                          |
| `instance_name`.evt.timeout                                | Integer | The event is changed to timeout "3".                                                                     |
| `instance_name`.evt.stat                                   | Integer | The event is changed to stat "2".                                                                        |
| `instance_name`.evt.dial_result                            | Integer | It checks if there is an error, if there is, the status is changed to dial error.                        |
| `instance_name`.evt.dtmfsent                               | Integer | It checks if there is not an error, if there is, the status is changed to dtmf send.                     |
| `instance_name_modem`.evt.pcm                              | Buffer  | The buff is sent to the corresponding cabin audio module.                                                |
| `instance_name_modem`.evt.bstat                            | Integer | The event is changed to Modem bstat "1".                                                                 |
| `instance_name_modem`.evt.dtmf                             | String  | The event is changed to Modem dtmf "7".                                                                  |
| `instance_name_modem`.evt.ring                             | Integer | The event is changed to Modem ring "8".                                                                  |
| `instance_name_slic`.evt.dtmf                              | Integer | The event is changed to Modem dtmf "7".                                                                  |
| `instance_name_slic`.evt.pcm                               | Integer | The buff is sent to the corresponding cabin audio module.                                                |
| `instance_name_slic`.evt.onhook                            | Integer | The event is changed if value is 1 on hook or if value is 0 off hook.                                    |
| `instance_name_audio`.evt.pcm                              | Integer | Depending on the state the publication cmd.pcm is sent to modem or slic                                  |
| `instance_name_audio`.evt.alarmbtn                         | Integer | Depending on the value received, one event or another will be activated. (0 emergency, 1 sos, 2 cancel). |
| `instance_name_audio`.evt.mod.stat                         | Integer | If the module is not in start it will go into respawn.                                                   |
| `instance_name_audio`.evt.battery.stat                     | Integer | Publish to set battery_stat to the corresponding cabin audio module.                                     |
| `instance_name_audiogen`.evt.started.`instance_name`       | Integer | The event is changed to Audiogen started audio "15".                                                     |
| `instance_name_audiogen`.evt.stopped.`instance_name`       | Integer | The event is changed to Audiogen stopped audio "16".                                                     |
| `instance_name_audiogen`.evt.started.`instance_name_modem` | Integer | The event is changed to Audiogen started modem "17".                                                     |
| `instance_name_audiogen`.evt.stopped.`instance_name_modem` | Integer | The event is changed to Audiogen stopped modem "18".                                                     |
| *cmd*                                                      |         |                                                                                                          |
| `instance_name`.cmd.mute_am                                | Integer | Publication if the value is 1 the calls are muted, if the value 0 they are unmuted.                      |
| `instance_name`.cmd.test                                   | Integer | Publication makes a test call.                                                                           |
| `instance_name_modem`.cmd.pcmsend                          | Buffer  | Publish send to buffer.                                                                                  | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name_slic`.cmd.pcmsend                           | Buffer  | Publish send to buffer.                                                                                  | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `audio[car_number - 1]`.cmd.pcmsend                        | Buffer  | Publish send to buffer.                                                                                  | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| *sys*                                                      |         |
| `sys.alarm.end_of_alarm.*`                                 | Integer | The alert of the selected cabin is disabled by means of the value                                        |
| `emergency_rtopic`                                         | String  | If the error flag is activated, the received message is stored as a state                                |

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

| Parameter                  | Description                                                                   | Default Value    |
| :------------------------- | :---------------------------------------------------------------------------- | :--------------- |
| **tty_cmd**                | Path of the TTY device for AT commands.                                       | `"/dev/ttyUSB0"` |
| **tty_pcm**                | Path of the TTY device for voice.                                             | `""`             |
| **init_str**               | Initial AT command to send to the modem.                                      | `"ATE0V1"`       |
| **rx_frm_size**            | Reception buffer size (in bytes).                                             | `800`            |
| **tx_frm_size**            | Transmission buffer size (in bytes).                                          | `320`            |
| **dtmf_threshold**         | DTMF detection threshold.                                                     | `-20`            |
| **dtmf_min_gain**          | Minimum gain (in dBs) to detect a DTMF.                                       | `-40.0`          |
| **dtmf_min_gap**           | Minimum gap in milliseconds between a DTMF and the previous one to detect it. | `85`             |
| **dtmf_twist**             | Gain of the high tone (in dBs).                                               | `-1.0`           |
| **dtmf_rtwist**            | Gain of the low tone (in dBs).                                                | `8.0`            |
| **spk_vol**                | Modem speaker gain (in dBs).                                                  | `7`              |
| **data_only**              | Use the modem only for data calls.                                            | `false`          |
| **regen_dtmf_duration_ms** | Duration in milliseconds of the regenerated tones sent with the modem.        | `100`            |

### Pubsub topics

| Topic                                       | Type    | Description                                                                                            | Flags                                |
| :------------------------------------------ | :------ | :----------------------------------------------------------------------------------------------------- | :----------------------------------- |
| *evt*                                       |         |
| `instance_name`.evt.timeout                 | Integer | Activate to ev_timeout                                                                                 |
| `instance_name`.evt.clcc_timeout            | Integer | Activate to ev_clcc_timeout                                                                            |
| `instance_name`.evt.stat                    | Integer | Activate to ev_stat                                                                                    |
| `instance_name`.evt.atrecv                  |         | Activate to ev_modem_str                                                                               |
| `instance_name`.evt.sms.part                | String  | Post to diferents sms                                                                                  |
| `instance_name`.evt.pcm                     | String  | Manage if there are different sms and send them ??                                                     | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.needpcm                 | int     | Publish send to size buffer                                                                            | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.stopped.`instance_name` |         | Send ok to rtopic                                                                                      |
| *cmd*                                       |         |
| `instance_name`.cmd.atsend                  | String  | Post command AT                                                                                        |
| `instance_name`.cmd.atsendraw               | String  | Post command AT                                                                                        |
| `instance_name`.cmd.datasend                | Buffer  | Post buffer                                                                                            |
| `instance_name`.cmd.hangup                  | String  | Activate to ev_hangub                                                                                  |
| `instance_name`.cmd.answer                  | String  | Activate to ev_answer                                                                                  |
| `instance_name`.cmd.qbstat                  |         | Post to stat modem                                                                                     |
| `instance_name`.cmd.dial                    | String  | Call the number indicated by parameter.                                                                |
| `instance_name`.cmd.call_blacklist_add      | String  | Number is added to a blacklist                                                                         |
| `instance_name`.cmd.call_blacklist_del      | String  | Number is deleted from a blacklist                                                                     |
| `instance_name`.cmd.agc_print               |         | If agc is enabled it will print the agc information                                                    |
| `instance_name`.cmd.pcmsend                 | Buffer  | Post to buffer                                                                                         |
| `instance_name`.cmd.sms                     | String  | Post to SMS                                                                                            |
| `instance_name`.cmd.divert                  | String  | Publish forward calls from a number, a json is passed with three parameters phone, period and oneshot. |
| `instance_name`.cmd.force_pub_state         |         | Force to publish the csq and the creg.                                                                 |
| `instance_name`.cmd.dtmfsend                | String  | Publishes the sent dtfm, through a configuration json that is obtained from the config                 |
| *sys*                                       |         |
| `sys.phone.cmd.divert`                      | String  | Publish forward calls from a number, a json is passed with three parameters phone, period and oneshot. |

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

| Parameter      | Description                                                                                                                                       | Default Value        |
| :------------- | :------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------- |
| **led_`name`** | Multiple configurations for multiple LEDs assigning a `name` for each one. The value is a string with de path of the led device to be controlled. | `""`                 |
| **switch_off** | A comma-separated string list with the `name`s of the LEDs to be turned off when stopping the instance.                                           | `"lan,wan,wlan,usb"` |


### Pubsub topics

| Topic                            | Type   | Description                                      |
| :------------------------------- | :----- | :----------------------------------------------- |
| *cmd*                            |        |
| `instance_name`.cmd.brightness.* | String | Turn the led on or off                           |
| `instance_name`.cmd.trigger.*    | String | Turn on or off the led with higher configuration |

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

| Parameter        | Description                                                                             | Default Value |
| :--------------- | :-------------------------------------------------------------------------------------- | :------------ |
| **timeout**      | Maximum time by default to execute commands (can be modified when executing a command). | `180`         |
| **max_in_size**  | Maximum number of bytes to write as command input.                                      | `131072`      |
| **max_out_size** | Maximum number of bytes to read from command output.                                    | `131072`      |

### Pubsub topics

| Topic                         | Type   | Description                                                                  |
| :---------------------------- | :----- | :--------------------------------------------------------------------------- |
| *evt*                         |        |
| `instance_name`.evt.stop_cmds |        | The process that was running is killed                                       |
| *cmd*                         |        |
| `instance_name`.cmd           | String | Executes the json that has been passed as a parameter and returns the result |

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

| Parameter             | Description                                                                                                                            | Default Value           |
| :-------------------- | :------------------------------------------------------------------------------------------------------------------------------------- | :---------------------- |
| **timeout**           | Time in seconds to wait after starting the instance to start trying to send the report. This allows for the other instances to start.  | `10`                    |
| **iface**             | Network interface to extract the MAC to include in the report.                                                                         | `"eth0"`                |
| **posturl**           | Address to send the POST with the report.                                                                                              | `"http://localhost:80"` |
| **max_retry_timeout** | Maximum time in seconds to wait between tries to send the report (it increases linearly at a reason of 10 seconds until this maximum). | `3600`                  |
### Pubsub topics

| Topic                        | Type   | Description                          |
| :--------------------------- | :----- | :----------------------------------- |
| *cmd*                        |        |
| `instance_name`.cmd.generate | String | Send a report of all system modules. |
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

| Parameter           | Description                                                                                          | Default Value                   |
| :------------------ | :--------------------------------------------------------------------------------------------------- | :------------------------------ |
| **ifaces**          | Comma-separated string list of interfaces to track.                                                  | `"eth0,eth1,wlan0,br-lan,ppp0"` |
| **clean_untracked** | Prioritize the tracked interfaces to the others of the system (moving the others to higher metrics). | `false`                         |
| **ping_list**       | Comma-separated string list of hosts to ping.                                                        | `"8.8.8.8,8.8.4.4"`             |
| **off_period**      | Interval in seconds to check the interfaces when no ping has succeeded.                              | `10.0`                          |
| **on_period**       | Interval in seconds to check the interfaces when a ping has succeeded.                               | `60.0`                          |
| **ping_path**       | Command to execute for the ping utility.                                                             | `"ping"`                        |
| **ping_tout**       | Timeout in seconds for the pings.                                                                    | `4.0`                           |
| **ip_path**         | Command to execute for the ip utility.                                                               | `"ip"`                          |
| **route_path**      | Command to execute for the route utility.                                                            | `"route"`                       |
| **awk_path**        | Command to execute for the awk utility.                                                              | `"awk"`                         |
| **head_path**       | Command to execute for the head utility.                                                             | `"head"`                        |
| **panic_tout**      | Time in seconds without ping to execute the panic command.                                           | `900`                           |
| **panic_cmd**       | Command to execute when the panic timeout expires (none if left empty).                              | `"reboot"`                      |
### Pubsub topics

| Topic                              | Type    | Description                                      |
| :--------------------------------- | :------ | :----------------------------------------------- |
| *evt*                              |         |
| `instance_name`.evt.ping_done      | String  | Wdinet received a response from a ping execution |
| `instance_name`.evt.get_route_done | String  | get the route                                    |
| `instance_name`.evt.set_route_done | String  | set the route                                    |
| `instance_name`.evt.stat           | String  | Exec module has changed state                    |
| `instance_name`.evt.timeout        | String  | Ping response timed out                          |
| `instance_name_exec`.evt.mod.stat  | String  | Exec module has changed state                    |
| `instance_name_n4m`.evt.secure     | Integer | safe_n4m mode is activated                       |
| *cmd*                              |         |
| `instance_name`.cmd.check_now      | String  | Connectivity is verified.                        |


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

| Parameter           | Description                                                                         | Default Value      |
| :------------------ | :---------------------------------------------------------------------------------- | :----------------- |
| **tty_path**        | Path of the modem TTY device.                                                       | `"/dev/ttyUSB0"`   |
| **tty_hwc**         | Enable TTY hardware flowcontrol.                                                    | `false`            |
| **timeout**         | Timeout in seconds to wait for AT command responses.                                | `5.0`              |
| **modem_init**      | Initial AT command to send to the modem.                                            | `"ATE0V1"`         |
| **pppd_path**       | Command to execute the pppd utility.                                                | `"pppd"`           |
| **pppd_params**     | Extra parameters to pass to pppd.                                                   | `"mtu 1450"`       |
| **pppd_wait_iface** | Time in seconds to wait for the interface to be ready after launching pppd.         | `25`               |
| **unit**            | Number of the desired interface to pass to pppd, used to name the interface (pppX). | `0`                |
| **metric**          | Desired metric of the interface to pass to pppd (see `wdinet` module).              | `30`               |
| **apn_file**        | Path of the JSON file to use as list of APNs.                                       | `"/etc/apns.json"` |
| **force_apn**       | Force the specified APN (if empty the file with the list of APNs is used).          | `""`               |
| **force_apn_user**  | User to use with the forced APN.                                                    | `""`               |
| **force_apn_pass**  | Password to use with the forced APN.                                                | `""`               |
| **panic_timeout**   | Time in seconds without internet through 3G to execute the panic command.           | `600`              |
| **panic_cmd**       | Command to execute when the panic timeout expires (none if left empty).             | `""`               |

### Pubsub topics

| Topic                               | Type    | Description                                          |
| :---------------------------------- | :------ | :--------------------------------------------------- |
| *evt*                               |         |
| `instance_name`.evt.stat            | Integer | changed stat                                         |
| `instance_name`.evt.recv            | String  | Received AT command                                  |
| `instance_name`.evt.pppd_recv       | String  | Received pppd log line (change state when connected) |
| `instance_name`.evt.timeout         | Integer | active timeout                                       |
| `instance_name`.evt.exec_response   | String  | change  state to M3G_ST_RUNNING_WAIT_IFACE           |
| `instance_name_wdinet`.evt.mod.stat | Integer | Wdinet module has changed state                      |
| `instance_name_modem`.evt.bstat     | Integer | Modem module has changed state                       |

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

| Parameter             | Description                                                                                                                                                                                                                                                    | Default Value                               |
| :-------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------ |
| **modem_dev**         | Modem module instance name from which to receive SMSs.                                                                                                                                                                                                         | `"modem"`                                   |
| **password**          | Password the SMS must contain to be valid.                                                                                                                                                                                                                     | `"22sms22"`                                 |
| **respond**           | Respond to all the SMSs (included those that don't requested a response).                                                                                                                                                                                      | `true`                                      |
| **report_parameters** | A map to specify the data to be returned by a SMS report. The keys correspond to a section (an instance name or `main`) in the report generated by the `report` module and the values correpond to the parameter of that section to be included in the report. | `{"main": "id", "console": "remote_conns"}` |
| **$`param_alias`**    | One alias for each parameter of the config to use with the SMSs, instead of writing the instance and the parameter, the alias can be used to save characters on the SMS.                                                                                       |

### Pubsub topics

| Topic                                      | Type    | Description              |
| :----------------------------------------- | :------ | :----------------------- |
| *evt*                                      |         |
| `instance_name_mode`.evt.sms               | String  | Send sms                 |
| `instance_name`.got_state                  | String  | Changed last_state_query |
| *sys*                                      |         |
| sys.alert.`instance_name`                  | String  | Send the Json of the sms |
| sys.cmd.cfg.set.`instance_name`            | String  | Send new config          |
| sys.cmd.cfg.get.`instance_name`            | String  | Get config               |
| sys.cmd.cfg.save                           | String  | Save config              |
| sys.alarm.end_of_alarm.sms.`instance_name` | Integer | remove cabin alarm       |

___

## Battery  (Deprecated)

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

| Parameter         | Description                                                                | Default Value  |
| :---------------- | :------------------------------------------------------------------------- | :------------- |
| **i2cbus**        | I2C device path.                                                           | `"/dev/i2c-0"` |
| **reload_period** | Interval in seconds to read and publish battery data.                      | `60`           |
| **vars**          | A list for each battery variable defining the thresholds and alert values. | `[]`           |

#### vars parameters

| Parameter       | Description                                                                                                           |
| :-------------- | :-------------------------------------------------------------------------------------------------------------------- |
| **name**        | Name of the variable (`"v"` for voltage, `"i"` for intensity, `"t"` for temperature and `"soc"` for state of charge). |
| **thold**       | Threshold to publish again the read data (from the last published value).                                             |
| **alert**       | Wheter to activate the alerts for the variable (`true`) or not (`false`).                                             |
| **alert_level** | *(only for `v`)* Limit value to generate the alert state.                                                             |
| **alert_safe**  | *(only for `v`)* Limit value to generate the alert recovery state.                                                    |
| **alert_max**   | *(only for `t`)* Limit value to generate the alert state.                                                             |
### Pubsub topics

| Topic                          | Type   | Description                                              | Flags |
| :----------------------------- | :----- | :------------------------------------------------------- | :---- |
| *evt*                          |        |
| `instance_name`.evt.`v_name`   | Double | Publish v value                                          |
| `instance_name`.evt.`i_name`   | Double | Publish i value                                          |
| `instance_name`.evt.`t_name`   | Double | Publish t value                                          |
| `instance_name`.evt.`soc_name` | Double | Publish soc value                                        |
| *cmd*                          |        |
| `instance_name`.cmd.query.`*`  | String | Publish the value of the parameter received in the path. |
| `instance_name`.cmd.resend     | String | publish the value of (v ,i ,t, soc)                      |
| `instance_name`.cmd.resend     | String | values are changed to 0 and public (v ,i ,t, soc)        |
| *sys*                          |        |
| sys.alert.`instance_name`      | String | Publish alert json                                       |

___
## Max17048  

### Description

This module is responsible of periodically checking the battery status, publishing the read values of the battery in the pubsub and generating alerts depending on the threshold values defined for each variable. Works with a BQ34Z100 module.

### Configuration example

    {
    ... other instances ...
            "max17048": {
        "autostart": true,
        "cell_num": 2,
        "i2cbus": "/dev/i2c-0",
        "log_level": "none",
        "max_respawn_delay": 60,
        "module": "max17048",
        "overvolt_alert": 8.5,
        "overvolt_safe": 8.3,
        "rcomp": 151,
        "reset_mV": 3000,
        "respawn": true,
        "soc_alert": 50,
        "soc_safe": 60,
        "undervolt_alert": 7,
        "undervolt_safe": 7.2
    }, 
    ... other instances ...
    }

### Specific module parameters

| Parameter  | Description      | Default Value  |
| :--------- | :--------------- | :------------- |
| **i2cbus** | I2C device path. | `"/dev/i2c-0"` |


#### parameters

| Parameter       | Description                                                                                                           |
| :-------------- | :-------------------------------------------------------------------------------------------------------------------- |
| **name**        | Name of the variable (`"v"` for voltage, `"i"` for intensity, `"t"` for temperature and `"soc"` for state of charge). |
| **thold**       | Threshold to publish again the read data (from the last published value).                                             |
| **alert**       | Wheter to activate the alerts for the variable (`true`) or not (`false`).                                             |
| **alert_level** | *(only for `v`)* Limit value to generate the alert state.                                                             |
| **alert_safe**  | *(only for `v`)* Limit value to generate the alert recovery state.                                                    |
| **alert_max**   | *(only for `t`)* Limit value to generate the alert state.                                                             |

### Pubsub topics

| Topic                           | Type    | Description                                                                                           |
| :------------------------------ | :------ | :---------------------------------------------------------------------------------------------------- |
| *evt*                           |         |
| `instance_name`.evt.soc         | String  | Process and send battery alerts.                                                                      |
| `instance_name_adc`.evt.voltage | Integer | Set value to adc. if greater than 16 true otherwise false.                                            |
| `instance_name`.evt.minutes     | Double  | Publish the remaining battery time.                                                                   |
| *cmd*                           |         |
| `instance_name`.cmd.query       | String  | publish the values vcell, soc and If the battery is not charging, publish the remaining battery time. |
| `instance_name`.cmd.rcomp       | Integer | Set value to rcomp.                                                                                   |
| *sys*                           |         |
| sys.alert.`instance_name.soc`   | String  |

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

| Parameter      | Description                                                  | Default Value     |
| :------------- | :----------------------------------------------------------- | :---------------- |
| **path**       | Path to watchdog device.                                     | `"/dev/watchdog"` |
| **interval**   | Time in seconds to wait between writes to watchdog.          | `1`               |
| **auto_open**  | Open the watchdog automatically when starting the instance.  | `true`            |
| **auto_close** | Close the watchdog automatically when stopping the instance. | `true`            |

### Pubsub topics

| Topic                           | Type    | Description                        |
| :------------------------------ | :------ | :--------------------------------- |
| *cmd*                           |         |
| `instance_name`.cmd.open        | String  | Open watchdog with current options |
| `instance_name`.cmd.close       | Integer | Close watchdog                     |
| `instance_name`.cmd.timeout.get | Integer | Get timeout to options             |
| `instance_name`.cmd.timeout.set | Integer | Set timeout                        |
| `instance_name`.cmd.options.get | Integer | Get options                        |

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

| Parameter                    | Description                                                                                     | Default Value          |
| :--------------------------- | :---------------------------------------------------------------------------------------------- | :--------------------- |
| **host**                     | Address of the VPN server to connect.                                                           | `"eu1.z.n4m.zone"`     |
| **port**                     | Port of the VPN server to connect.                                                              | `7744`                 |
| **user**                     | User to authenticate.                                                                           | `"user"`               |
| **password**                 | Password to authenticate.                                                                       | `"password"`           |
| **if_name**                  | Name of the created interface.                                                                  | `"n4m%d"`              |
| **fail_cmd_error**           | Treat as an error the execution of commands with a return value different to zero.              | `false`                |
| **restart_on_default_route** | Restart the instance when a change on the default route is found through the `wdinet` instance. | `true`                 |
| **ip_cmd**                   | Command to execute the ip utility.                                                              | `"/usr/sbin/ip"`       |
| **iptables_cmd**             | Command to execute the iptables utility.                                                        | `"/usr/sbin/iptables"` |

### Pubsub topics

| Topic                                    | Type    | Description                                                                                                               |
| :--------------------------------------- | :------ | :------------------------------------------------------------------------------------------------------------------------ |
| *evt*                                    |         |
| `instance_name`.evt.stat                 | Integer | Set new stat                                                                                                              |
| `instance_name`.evt.timeout              | String  | Active timeout                                                                                                            |
| `instance_name_wdinet`.evt.def_iface     | String  | Restart module                                                                                                            |
| `instance_name_modem`.evt.bstat          | Integer | Set bstat                                                                                                                 |
| `instance_name_modem`.evt.cops           | Integer | Set cops                                                                                                                  |
| `instance_name`.evt.secure               | Integer | it is 1 when GSM secure activated and working over 2G. Stopping VPN and it's 0 when GSM secure not activated, go to open. |
| `instance_name`.evt.nl.dellink.`ifname`  | String  | Link down. Stop (not used).                                                                                               |
| `instance_name`.evt.nl.delladdr.`ifname` | String  | IP address removed. Stop (not used).                                                                                      |
| *cmd*                                    |         |
| `instance_name`.cmd.addmetainfo          | String  | Add info to ping (not used)                                                                                               |
| *sys*                                    |         |
| sys.evt.realtime                         | String  | Add info to ping (not used)                                                                                               |
| *other*                                  |         |
| `instance_name`.status.`ifname`          | String  | Publish a json with the status of the vpn.                                                                                |

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

| Parameter   | Description                                       | Default Value         |
| :---------- | :------------------------------------------------ | :-------------------- |
| **devpath** | Path to the device to be watched by the instance. | `"/dev/input/button"` |

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

| Parameter      | Description                                                                                                           | Default Value |
| :------------- | :-------------------------------------------------------------------------------------------------------------------- | :------------ |
| **alert_path** | Path of the pubsub to subscribe for receiving alerts (`sys.alert.<alert_path>`).                                      | `""`          |
| **sms_to**     | Phone number to send the SMS to. Not send by SMS if empty.                                                            | `""`          |
| **modem_dev**  | Instance name of the `modem` module to send the SMSs.                                                                 | `"modem"`     |
| **url_post**   | Address to send the alert through POST. Not sent by POST if empty.                                                    | `""`          |
| **timeout**    | Time in seconds to wait between tries to send the alert. It increments exponentially.                                 | `3600`        |
| **max_ttl**    | Maximum life time in seconds for the alert, when this time expires and the alert has not been notified, it's deleted. | `86400`       |
### Pubsub topics

| Topic                         | Type    | Description                                                   |
| :---------------------------- | :------ | :------------------------------------------------------------ |
| *evt*                         |         |
| `instance_name`.evt.stat      | String  | Call run machine                                              |
| `instance_name`.evt.mod.stat  | Integer | If the value is 1, is_modem_started is activated else disable |
| *cmd*                         |         |
| `instance_name_modem`.cmd.sms | String  | The alert is sent by sms                                      |
| *sys*                         |         |
| sys.alert.`alert_path`        | String  | Alert is saved and managed                                    |

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

| Parameter             | Description                                                               | Default Value |
| :-------------------- | :------------------------------------------------------------------------ | :------------ |
| **type**              | Type of event (/usr/include/linux/input-event-codes.h).                   | `255`         |
| **code**              | Key code (/usr/include/linux/input-event-codes.h).                        | `255`         |
| **keypress_min_secs** | Minimum number of seconds with the button pressed to execute the actions. | `0`           |
| **keypress_min_secs** | Maximum number of seconds with the button pressed to execute the actions. | `60`          |
| **actions**           | JSON string with a list of the actions as defined in the description.     | `"[]"`        |
### Pubsub topics

| Topic                               | Type                  | Description                                          | FlaGS          |
| :---------------------------------- | :-------------------- | :--------------------------------------------------- | :------------- |
| *evt*                               |                       |                                                      |
| `instance_name_evdev`.evt.mod.stat  | Integer               | Set new stat                                         |
| `instance_name_evdev`.qstat         | Integer               | Publish qstat value                                  | MSG_FL_INSTANT |
| `instance_name`.evt.btn.`type.code` | Integer               | Publish type & code (not used)                       |
| `action_topic`                      | Integer/Double/String | Post a value depending on the type of action (i,f,s) |
| *sys*                               |                       |
| sys.evdev.`type.code`               | String                | Publish the information on evt.btn                   |
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

### Pubsub topics

| Topic                                | Type    | Description                                                                                        |
| :----------------------------------- | :------ | :------------------------------------------------------------------------------------------------- |
| *evt*                                |         |
| `instance_name_modem`.evt.csq        | Integer | The 3g led is evaluated, depending on the value the led is of one color or another.                |
| `instance_name_modem`.evt.creg       | Integer | The 3g led is evaluated, depending on the value the led is of one color or another.                |
| `instance_name_modem`.evt.cops       | Integer | The 3g led is evaluated, depending on the value the led is of one color or another.                |
| `instance_name_batery`.evt.soc_panic | Integer | if low level of battery, enable led else safe level of battery, disable led.                       |
| `instance_name_gateway`.evt.stat     | Integer | Depending on the state of the gateway, one publication or another is sent.                         |
| `instance_name_telealarm`.evt.stat   | Integer | Depending on the state of the telealarm, one publication or another is sent.                       |
| *cmd*                                |         |
| `instance_name`.cmd.working          | Integer | Set a timer and post *cmd.trigger, cmd.brightness* with a setting                                  |
| `instance_name`.cmd.ok               | Integer | Set a timer and post *cmd.trigger, cmd.trigger.`leds`.delay_on/off, cmd.brightness* with a setting |
| `instance_name_leds`.cmd.brightness  | String  |
| `instance_name`.cmd.nok              | Integer | Post *cmd.trigger, cmd.brightness* with a setting                                                  |
| sys.cmd.cfg.set                      | Integer | Activates in morse mode if not on call, outgoing, working                                          |
| sys.cmd.cfg.set.*                    | Integer | Activates in morse mode if not on call, outgoing, working                                          |

___

## Openwrt-cfg

### Description

This module is reponsible of handling configurations of the underlying openWRT operating system. Mostly handles networking configurations.

### Configuration example

    {
        ... other instances ...
        "openwrt-cfg": {
            "module": "openwrt-cfg",
            "autostart": true,
            "log_level": "none",
            "respawn": true,
            "max_respawn_delay": 60,
            "exec_instance": "exec",
            "enable_wifi": true
	},
        ... other instances ...
    }

### Specific module parameters

| Parameter                                    | Description                                                                                 | Default Value     |
| :------------------------------------------- | :------------------------------------------------------------------------------------------ | :---------------- |
| **exec_instance**                            | Instancia del módulo exec a utilizar                                                        | `exec`            |
| **dependent_modules**                        | Módulos necesarios a reiniciar al aplicar un cambio                                         | `3g`              |
| **lan_protocol**                             | Protocolo a utilizar para el puerto LAN                                                     | `static`          |
| **lan_phy_name**                             | Interfaz física a utilizar para el puerto LAN                                               | `eth0`            |
| **lan_ip**                                   | Dirección IPv4 para el puerto LAN                                                           | `192.168.255.1`   |
| **lan_netmask**                              | Mascara IPv4 para el puerto LAN                                                             | `255.255.255.0`   |
| **lan_gateway**                              | Puerta de enlace IPv4 para el puerto LAN                                                    | `-`               |
| **lan_broadcast**                            | Broadcast IPv4 para el puerto LAN                                                           | `192.168.255.255` |
| **lan_custom_dns**                           | Utilizar servidor DNS proprio para el puerto LAN                                            | `-`               |
| **lan_enable_dhcp**                          | Habilitar servidor DHCP para el puerto LAN                                                  | `true`            |
| **wan_protocol**                             | Protocolo a utilizar para el puerto WAN                                                     | `dhcp`            |
| **wan_phy_name**                             | Interfaz física a utilizar para el puerto WAN                                               | `eth1`            |
| **wan_ip**                                   | Dirección IPv4 para el puerto WAN                                                           | `192.168.1.2`     |
| **wan_netmask**                              | Mascara IPv4 para el puerto WAN                                                             | `255.255.255.0`   |
| **wan_gateway**                              | Puerta de enlace IPv4 para el puerto WAN                                                    | `192.168.1.1`     |
| **wan_broadcast**                            | Broadcast IPv4 para el puerto WAN                                                           | `192.168.1.255`   |
| **wan_custom_dns**                           | Utilizar servidor DNS proprio para el puerto WAN                                            | `-`               |
| **enable_wifi**                              | Habilitar el wifi de la primera interfaz disponible                                         | `true`            |
| **set_wan_as_lan**                           | Habilitar el puerto wan como lan                                                            | `false`           |
| **set_priority_ppp_over_wan**                | Priorizar la salida de datos por PPP en lugar de WAN                                        | `false`           |
| **max_enable_time_wifi_offline_minutes**     | Tiempo máximo de funcionamiento del punto de acceso WiFi Offline                            | `240`             |
| **periodic_check_time_wifi_offline_minutes** | Tiempo entre comprobaciones si hay clientes conectados al WiFi Offline                      | `30`              |
| **radio_rx_power**                           | Potencia de la radio de recepción dBm (-1 auto)                                             | `-1`              |
| **radio_tx_power**                           | Potencia de la radio de transmisión (-1 auto)                                               | `-1`              |
| **radio_distance**                           | Distancia en metros del cliente más alejado (-1 auto)                                       | `-1`              |
| **radio_channel**                            | Canal WiFi a utilizar                                                                       | `auto`            |
| **radio_country_code**                       | Código de país de dos letras. Afecta los canales disponibles y las potencias de transmisión | `ES`              |
| **radio_hw_mode**                            | Protocolo WiFi a utilizar                                                                   | `11g`             |
| **radio_ht_mode**                            | Anchura del canal                                                                           | `HT20`            |
| **wifi_device**                              | Interfaz física WiFi a utilizar                                                             | `radio0`          |
| **wifi_network**                             | Regla de firewall a aplicar                                                                 | `lan`             |
| **wifi_mode**                                | Modo de operación.                                                                          | `ap`              |
| **wifi_ssid**                                | Nombre de la red WifFi                                                                      | `GSR`             |
| **wifi_hide_ssid**                           | Esconder el nombre de la red                                                                | `false`           |
| **wifi_encryption**                          | Tipo de cifrado                                                                             | `wpa2+ccmp`       |
| **wifi_key**                                 | Contraseña de la red WiFi                                                                   | `-`               |
| **wifi_radius_server**                       | IP del servidor de autenticación RADIUS                                                     | `10.72.72.3`      |
| **wifi_radius_port**                         | Puerto del servidor de autenticación RADIUS                                                 | `1812`            |
| **wifi_radius_secret**                       | Secreto comprartido con el servidor de autenticación RADIUS                                 | `nayarsystems`    |
| **wifi_offline_device**                      | Interfaz física WiFi a utilizar para la WiFi Offline                                        | `radio0`          |
| **wifi_offline_network**                     | Regla de firewall a aplicar para la WiFi Offline                                            | `lanOffline`      |
| **wifi_offline_mode**                        | Modo de operación.                                                                          | `ap`              |
| **wifi_offline_ssid**                        | Nombre de la red WifFi para la WiFi Offline                                                 | `-`               |
| **wifi_offline_hide_ssid**                   | Esconder el nombre de la red para la WiFi Offline                                           | `false`           |
| **wifi_offline_encryption**                  | Tipo de cifrado para la WiFi Offline                                                        | `psk-mixed`       |
| **wifi_offline_disable**                     | Deshabilitar red WiFi para la WiFi Offline                                                  | `true`            |
| **wifi_offline_key**                         | Contraseña de la red WiFi para la WiFi Offline                                              | `12345678`        |

### Pubsub topics

| Topic                                   | Type    | Description                                                 |
| :-------------------------------------- | :------ | :---------------------------------------------------------- |
| *evt*                                   |         |
| `instance_name_exec`.evt.mod.stat       | Integer | WAN and WLAN settings are modified. Safe mode is activated. |
| *cmd*                                   |         |
| `instance_name`.cmd.setwifi             | Integer | Enable WIFI                                                 |
| `instance_name`.cmd.enable_wifi         | Integer | Enable WIFI                                                 |
| `instance_name`.cmd.set_wan_as_lan      | Integer | Enable LAN                                                  |
| `instance_name`.cmd.enable_wifi_offline | Integer | Disable WIFI                                                |
| `instance_name`.cmd.set_ppp_over_wan    | Integer | Set config to WAN                                           |
| `instance_name`.cmd.get_model_ehci      | String  | Check gsr version and post version                          |
| `instance_name`.cmd.get_udev_rules      | String  | Modify rules udev                                           |
| `instance_name`.cmd.reload_udev_rules   | Integer | Update rules udev                                           |


## Modbus RTU

### Description

This module is reponsible of interpreting the serial modbus protocol from to GSR.

### Configuration example

    {
        ... other instances ...
        "modbus": {
            "module": "modbus_rtu",
            "log_level": "none",
            "autostart": true,
            "respawn": true
            "max_respawn_delay": 60,
            "serial_instance": "serial",
            "slave_id":1
	    },
        ... other instances ...
    }

### Specific module parameters

| Parameter                  | Description                                             | Default Value |
| :------------------------- | :------------------------------------------------------ | :------------ |
| **serial_instance**        | Serial instance to use for receiving raw data.          | `serial`      |
| **slave_id**               | ID of the modbus slave                                  | `1`           |
| **max_reply_delay_ms**     | Tymeout for receiving the data from the bus             | `1500`        |
| **request_proc_queue_len** | Internal queue for processing serial events from serial | 128           |

### Pubsub topics examples

READ FUNCTIONS

`instance_name`.cmd.read.coils:{"addr":304, "nb":1}
`instance_name`.cmd.read.discrete_input:{"addr":452, "nb":1}
`instance_name`.cmd.read.holding_registers:{"addr":2057,"nb":1}
`instance_name`.cmd.read.input_registers:{"addr":264, "nb":1}

WRITE FUNCTIONS

`instance_name`.cmd.write.single_coil:{"addr":304, "value":0}
`instance_name`.cmd.write.single_holding_register:{"addr":2060, "value":1}
`instance_name`.cmd.write.multiple_coils:{"addr":304, "values":[1,0,1]}
`instance_name`.cmd.write.multiple_coils:{"addr":304, "values":[1,0,1,0,1,0,1,0]}
`instance_name`.cmd.write.multiple_holding_registers:{"addr":2060, "values":[0005,0004,0008]}

MONITOR FUNCTIONS

`instance_name`.cmd.monitor.coils:{"addr":123, "nb":1, "period_ms":1000, "tag":"vvvf_status"}
`instance_name`.evt.vvvf_status
`instance_name`.cmd.monitor.stop:{"tag":"vvvf_status"}

`instance_name`.cmd.monitor.holding_registers:{"addr":2060, "nb":10, "period_ms":1000, "tag":"frequency"}
`instance_name`.evt.frequency
`instance_name`.cmd.monitor.stop:{"tag":"frequency"}

### Pubsub topics

| Topic                                                | Type    | Description                                                  | Flags                                |
| :--------------------------------------------------- | :------ | :----------------------------------------------------------- | :----------------------------------- |
| *cmd*                                                |         |                                                              |
| `instance_name`.cmd.read.coils                       | String  | Depending on the json read one addr or another               |
| `instance_name`.cmd.read.discrete_input              | String  | Depending on the json read one addr or another               |
| `instance_name`.cmd.read.holding_registers           | String  | Depending on the json read one addr or another               |
| `instance_name`.cmd.read.input_registers             | String  | Depending on the json read one addr or another               |
| `instance_name`.cmd.write.single_coil                | String  | Depending on the json write one addr or another              |
| `instance_name`.cmd.write.single_holding_register    | String  | Depending on the json write one addr or another              |
| `instance_name`.cmd.write.multiple_coils             | String  | Depending on the json write one addr or another              |
| `instance_name`.cmd.write.multiple_holding_registers | String  | Depending on the json write one addr or another              |
| `instance_name`.cmd.monitor.coils                    | String  | Depending on the json it monitors one addr or another        |
| `instance_name`.cmd.monitor.stop                     | String  | Del event                                                    |
| `instance_name`.cmd.monitor.holding_registers        | String  | Depending on the json it monitors one addr or another        |
| `instance_name_serial`.cmd.write.                    | Buffer  | Publish sent to buffer                                       | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| *evt*                                                |         |
| `instance_name_serial`.evt.recv                      | Buffer  | Read buffer and post frame in `modbus_frame`.evt.recv.frame. |
| `instance_name_serial`.evt.mod.stat                  | Integer | Modify state                                                 |
___


## Events

### Description

This module is reponsible of sending events from GSR to URL entrypoing. The data is compressed using gz.

### Configuration example

    {
        ... other instances ...
        "events": {
            "module": "events",
            "log_level": "none",
            "autostart": true,
            "respawn": true
            "max_respawn_delay": 60,
            "parse_json": false,
            "post_url": "http://localhost:8000/",
            "filter_json": "",
            "send_max_elements": 100,
            "send_period_s": 3600,
            "changes_only": true,
            "flush_paths": "test.flush,a.fuuu",
            "topic": "test.flush.json",
            "info_str": "{\"element_id\": \"XXXXX\"}"
	    },
        ... other instances ...
    }

### Specific module parameters

| Parameter             | Description                                                                         | Default Value |
| :-------------------- | :---------------------------------------------------------------------------------- | :------------ |
| **parse_json**        | Specify if the data content is JSON serialized                                      | `false`       |
| **filter_json**       | Specify how the keys to search in the JSON, splited by `,`. Example `id,name,value` | `-`           |
| **post_url**          | URL entypoint of the data to be processed                                           | `-`           |
| **send_max_elements** | Number of elements to store before sending the events.                              | `1024`        |
| **send_period_s**     | Time to wait before sending the events in seconds.                                  | `3600`        |
| **changes_only**      | Specify if required to store only different data values.                            | `false`       |
| **flush_paths**       | Obelisk paths in which forces to send the actual data stored                        | `-`           |
| **topic**             | Obelisk topic to subscribe and monitor it's values                                  | `-`           |
| **info_str**          | Extra data for the entrypoing.                                                      | `-`           |
| **notify_event_path** | Path to send a notification that an event was sent.                                 | `-`           |
### Pubsub topics

| Topic                     | Type   | Description                    |
| :------------------------ | :----- | :----------------------------- |
| *cmd*                     |        |
| `instance_name`.cmd.flush | String | Send events.                   |
| `topic`                   | String | Event is received and treated. |

___

## Monitor

### Description

This module is reponsible for monitoring states of other modules.

### Configuration example

    {
        ... other instances ...
        "monitor": {
            "module": "monitor",
            "log_level": "none",
            "autostart": true,
            "respawn": true
            "max_respawn_delay": 60,
            "instances_uptime_monitor": "some_events,serial,serial@",
            "instances_uptime_monitor_all": false
	    },
        ... other instances ...
    }

### Specific module parameters

| Parameter                        | Description                                                                                                                                                                      | Default Value |
| :------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ |
| **instances_uptime_monitor**     | Specify modules name to monitor splited by `,`. If an `@` is added at the end of the modules it will monitor all the modules wit the format `<instance_name>@<autogenerated_id>` | `-`           |
| **instances_uptime_monitor_all** | Monitor all modules in the configuration file.                                                                                                                                   | `false`       |

### Pubsub topics examples

`instance_name`.evt.uptime.serial@adsfasdf:0
`instance_name`.evt.uptime.serial@bbbbbbbb:7
`instance_name`.evt.uptime.serial:15
`instance_name`.evt.uptime.some_events:10

### Pubsub topics

| Topic                                | Type    | Description                                                                                           |
| :----------------------------------- | :------ | :---------------------------------------------------------------------------------------------------- |
| *evt*                                |         |
| `instance_name`.evt.cfg.get          | String  | Get config ,  Add/delete instances from monitor map and if it was the starting, subscribe to reloads. |
| `instance_name_monitor`.evt.mod.stat | Integer | Publish to evt.publish.instance depending on the parameter received                                   |
| *cmd*                                |         |
| sys.cmd.reload                       | String  | Ask for the config again on system reloads                                                            |
___


## P100

### Description

The P100 protocol is a standard DTMF data communication protocol implemented by several remote alarms, used for test and/or alarm calls.
### Configuration example

    {
        ... other instances ...
        "p100": {
            "audiogen_instance": "audiogen",
            "autostart": true,
            "car_number1": 13,
            "car_number2": 3,
            "car_number3": 0,
            "car_number4": 0,
            "id": 13,
            "log_level": "none",
            "max_respawn_delay": 60,
            "modem_instance": "modem",
            "module": "p100",
            "respawn": true
        },
        ... other instances ...
    }

### Specific module parameters

| Parameter            | Description                            | Default Value |
| :------------------- | :------------------------------------- | :------------ |
| **car_number_id[0]** | id fo carnumber 1                      | `-1`          |
| **car_number_id[1]** | id fo carnumber 2                      | `-1`          |
| **car_number_id[2]** | id fo carnumber 3                      | `-1`          |
| **car_number_id[3]** | id fo carnumber 4                      | `-1`          |
| **call_timeout**     | Timeout for max calls duration         | `1800`        |
| **transmit_timeout** | Timeout for init tone transmit/receive | `0.4`         |


### Pubsub topics

| Topic                                                      | Type    | Description                                         |
| :--------------------------------------------------------- | :------ | :-------------------------------------------------- |
| *evt*                                                      |         |
| `instance_name`.evt.stat                                   | String  | Active ev_stat                                      |
| `instance_name`.evt.dtmf                                   | String  | Set dtmf and active ev_dtmf                         |
| `instance_name_audiogen`.evt.stopped.`instance_name_modem` | String  | Set stat to  MP100_WAITING_INIT_TONE                |
| *cmd*                                                      |         |
| `instance_name`.cmd.emerg                                  | String  | Make a emerg call                                   |
| `instance_name`.cmd.emerg_no_voice                         | Integer | make a call without voice                           |
| `instance_name`.cmd.autotest                               | Integer | Make a test call                                    |
| `instance_name`.cmd.failure                                | Integer | Set failure and set stat to MP100_WAITING_INIT_TONE |
| `instance_name`.cmd.cancel                                 | String  | Set stat to  MP100_WAITING_START                    |
___

## SIP

### Description

The SIP client is to be able to make emergency calls using only data.

### Configuration example

    {
        ... other instances ...
        "sip": {
            "alert_timeout": 5,
            "autostart": false,
            "client_uri": "sip:user@IP",
            "jbuf_max_pkts": 20,
            "jbuf_min_pkts": 5,
            "local_ip": "10.100.X.X",
            "log_level": "info",
            "max_respawn_delay": 10,
            "media_enc": "",
            "module": "sip",
            "options_errors_max": 2,
            "options_send_period": 0,
            "options_send_timeout": 5,
            "password": "*******",
            "reg_expiration": 100,
            "respawn": true,
            "rx_dtmf_min_gain": -100,
            "rx_dtmf_min_gap": 0,
            "rx_dtmf_rfc": false,
            "rx_dtmf_rtwist": -1,
            "rx_dtmf_threshold": -100,
            "rx_dtmf_twist": -1,
            "rx_rtp_burst_delta": 1120,
            "rx_rtp_burst_eob": 960,
            "server_address": "10.X.X.X",
            "slic_instance": "slic",
            "transport": "udp",
            "tx_dtmf_gain": -20,
            "tx_dtmf_off_time": 50,
            "tx_dtmf_on_time": 80,
            "tx_dtmf_rfc": false,
            "tx_dtmf_twist": 0,
            "username": "******"
        },
      ... other instances ...
    }

### Specific module parameters

| Parameter              | Description                                                                                                                 | Default Value |
| :--------------------- | :-------------------------------------------------------------------------------------------------------------------------- | :------------ |
| **server_address**     | SIP server where SIP client will request its registration                                                                   | ``            |
| **username**           | SIP client username                                                                                                         | ` `           |
| **password**           | SIP client password                                                                                                         | ` `           |
| **client_uri**         | SIP client UR                                                                                                               | ` `           |
| **local_ip**           | IP to be used by SIP and RTP sockets                                                                                        | ` `           |
| **media_enc**          | RTP encryption options                                                                                                      | ` `           |
| **transport**          | SIP Transport protocol: UDP, TCP, TLS                                                                                       | `udp `        |
| **tx_dtmf_rfc**        | 'true': Only notify DTMF reception if it has been detected by RFC 2833; 'false': Notify also the receptions of inband DTMFs | `false`       |
| **tx_dtmf_rfc**        | 'true': Transmit DTMF using RFC 2833; 'false': Transmit DTMF inband                                                         | `false`       |
| **tx_dtmf_on_time**    | Parameter for the inband DTMF generator: on_time value of audiogen cmd.play                                                 | `80 `         |
| **tx_dtmf_off_time**   | Parameter for the inband DTMF generator: off_time value of audiogen cmd.play                                                | `50 `         |
| **tx_dtmf_gain**       | Parameter for the inband DTMF generator: level value of audiogen cmd.play                                                   | ` -20`        |
| **tx_dtmf_twist**      | Parameter for the inband DTMF generator: twist value of audiogen cmd.play                                                   | ` 0`          |
| **rx_dtmf_threshold**  | Parameter for the inband DTMF detector: threshold parameter of the spandsp library                                          | ` -100`       |
| **rx_dtmf_twist**      | Parameter for the inband DTMF detector: twist parameter of the spandsp library                                              | ` -1`         |
| **rx_dtmf_rtwist**     | Parameter for the inband DTMF detector: rtwist parameter of the spandsp library                                             | ` -1`         |
| **rx_dtmf_min_gap**    | Parameter for the inband DTMF detector: minimum ms between detected DTMFs                                                   | ` 30`         |
| **rx_dtmf_min_gain**   | Parameter for the inband DTMF detector: minimum gain (dB)                                                                   | ` -30`        |
| **rx_rtp_burst_delta** | Minimum gap (ms) to consider a new audio burst has been detected                                                            | `1120`        |
| **reg_expiration**     | ms without receiving RTP packets to consider the end of the last audio burst                                                | `3600`        |
| **rx_rtp_burst_eob**   | ms without receiving RTP packets to consider the end of the last audio burst                                                | `960`         |
| **jbuf_min_pkts**      | Minimum number of RTP packets in the jitter buffer                                                                          | ``            |
| **jbuf_max_pkts**      | SIP_JBUF_MIN_PKTS_DESC                                                                                                      | ``            |



### Pubsub topics

| Topic                                                | Type    | Description                      | Flags               |
| :--------------------------------------------------- | :------ | :------------------------------- | :------------------ |
| *evt*                                                |         |                                  |
| `instance_name_audiogen`.evt.stopped.`instance_name` | String  | Set stat to  MP100_WAITING_START |
| `instance_name`.evt.pcm.`instance_name`              | Buffer  | Publish send to buffer           | MSG_FL_NONRECURSIVE |
| `instance_name`.evt.needpcm.`instance_name`          | Integer | Publish send to size buffer      | MSG_FL_NONRECURSIVE |

*cmd* | | |
`instance_name`.cmd.dial | String | Do call
`instance_name`.cmd.hangup | Integer | Hang up call
`instance_name`.cmd.dtmfsend | Integer | dtmfd is sent
`instance_name`.cmd.qbstat | Integer | Publish syayus in *evt.bstat*
`instance_name`.cmd.pcmsend | Buffer | Publish send to size buffer
`instance_name`.cmd.stop | String | stop mudule

___

## SMART

### Description

This is the module that handles all the devices from the Smart Control Platform.

### Configuration example

    {
        ... other instances ...
        "smart": {
        "module": "smart",
        "serial_instance": "serial",
        "can_instance": "socketcan",
        "modbus_instance": "modbus",
        "type": "liftcontroller",
        "element": "otis2000"
        },
      ... other instances ...
    }

### Specific module parameters

| Parameter           | Description                                                 | Default Value |
| :------------------ | :---------------------------------------------------------- | :------------ |
| **type**            | Element type to use. {liftcontroller, escalatorcontroller } | ` `           |
| **name**            | Element model to use                                        | ` `           |
| **serial_instance** | Instance of the serial module to use                        | ` `           |
| **can_instance**    | Instance of the socketcan module to use                     | ` `           |
| **modbus_instance** | Instance of the modbus module to use                        | ` `           |



### Pubsub topics

| Topic                                    | Type             | Description                                                                                |
| :--------------------------------------- | :--------------- | :----------------------------------------------------------------------------------------- |
| *evt*                                    |                  |
| `instance_name_serial`.evt.recv          | String           | A frame is received and passed to its instance                                             |
| `instance_name_serial`.evt.mod.stat      | Integer          | If the value is 1 it is stored, and it is activated                                        |
| `instance_name_can`.evt.frame            | String           | A frame is received and passed to its instance                                             |
| `instance_name_can`.evt.mod.stat         | Integer          | If the value is 1 it is stored, and it is activated                                        |
| `instance_name_modbus`.evt.frame         | String           | A frame is received and passed to its instance                                             |
| `instance_name_modbus`.evt.mod.stat      | Integer          | If the value is 1 it is stored, and it is activated                                        |
| `instance_name`.receive.evt.state        | String           | stores the state before sending the event                                                  |
| *cmd*                                    |                  |
| `instance_name`.cmd.telecontrol.activate | Integer          | If the value is greater than 0, the remote control is activated, if not, it is turned off. |
| `instance_name`.cmd.telecontrol.keypress | Integer / Buffer | Sends a keypress for telecontrol                                                           |
| `instance_name`.cmd.telemetry.status     | Integer          | stores the state                                                                           |
___

## TRIGGER

### Description

This module triggers a publication depending on the the given input topics values

### Configuration example

    {
        ... other instances ...
        "relay_trigger": {
            "module": "trigger",
            "inputs": [
                {
                    "topic": "modem.evt.linkstatus"
                }
            ],
            "trigger_min_secs": 10,
            "trigger_period": 1,
            "trigger_topic_true": "relay.cmd.off",
            "trigger_topic_false": "",
            "trigger_sticky": true
        }
      ... other instances ...
    }

### Specific module parameters

| Parameter               | Description                                                                          | Default Value |
| :---------------------- | :----------------------------------------------------------------------------------- | :------------ |
| **trigger_topic_true**  | Action to take when the condition is true                                            | ` `           |
| **trigger_topic_false** | Action to take when the condition is false                                           | ` `           |
| **trigger_min_secs**    | Time to perform the action                                                           | `0. `         |
| **trigger_period**      | Time to check                                                                        | `0. `         |
| **condition_func**      | action to evaluate, or or and                                                        | `and `        |
| **trigger_sticky**      | Post to sticky                                                                       | ` `           |
| **inputs**              | subscribes to the topic, this has two parameters, topic and not [{topic, not},{}...] | `[] `         |
| **inputs_input_topic**  | Subscribe to the topic                                                               | ` `           |
| **not**                 | Deny the topic                                                                       | ` `           |



### Pubsub topics

| Topic         | Type   | Description                                                                                    |
| :------------ | :----- | :--------------------------------------------------------------------------------------------- |
| current_topic | String | depending on the configuration, when the topic is received, the configured publication is sent |

___

## USBRELAY

### Description

Implements a pubsub interface to communicate with a usb relay
### Configuration example

    {
        ... other instances ...
        "relay": {
                "autostart": true,
                "module": "usbrelay",
                "serial_instance": "serial",
                "set_period": 2
            },
      ... other instances ...
    }

### Specific module parameters

| Parameter           | Description                                       | Default Value |
| :------------------ | :------------------------------------------------ | :------------ |
| **set_period**      | value to launch the timer to activate the relay   | `60. `        |
| **address**         | address to send the packet by serial to the relay | ` 0x01`       |
| **serial_instance** | Instance of the serial module to use              | `serial `     |


### Pubsub topics

| Topic                   | Type    | Description          |
| :---------------------- | :------ | :------------------- |
| `instance_name`.cmd.on  | Integer | activate the relay   |
| `instance_name`.cmd.off | Integer | deactivate the relay |

___


## VMODEM

### Description

Module that initializes a virtual modem
### Configuration example

    {
        ... other instances ...
        "vmodem": {
            "autostart": true,
            "log_level": "debug",
            "max_respawn_delay": 30,
            "module": "vmodem",
            "tty_type": "tty",
            "tty_path": "/dev/ttyUSB0",
            "baudrate": 38400,
            "databits": 8,
            "stopbits": 1,
            "parity": "NONE",
            "hwfc": false,
            "swfc": false,
            "conn_str": "CONNECT 38400",
            "listen_ip": "127.0.0.1",
            "listen_port": 5555
        }
      ... other instances ...
    }

### Specific module parameters

| Parameter       | Description                            | Default Value     |
| :-------------- | :------------------------------------- | :---------------- |
| **tty_type**    | TTY type (TTY, PTY, SOCK, ...)         | `tty `            |
| **tty_path**    | TTY device path                        | `/dev/ttyUSB0`    |
| **tty_wdt**     | TTY WatchDog timer                     | `0 `              |
| **ign_cmd**     | TTY flag to ignore AT commands         | `false`           |
| **dtr_mode**    | DTR mode 0=0ff 1=on 2=DCD              | `0 `              |
| **sync_delay**  | TTY sync_delay in milliseconds         | `0 `              |
| **baudrate**    | Port baudrate                          | `9600`            |
| **databits**    | databits                               | `0 `              |
| **parity**      | parity                                 | `NONE`            |
| **stopbits**    | stopbits                               | `0`               |
| **tty_hwfc**    | tty hardware flow contro               | `false`           |
| **tty_swfc**    | tty software flow control              | `false`           |
| **listen_ip**   | Listen ip                              | `127.0.0.1`       |
| **listen_port** | Listen port                            | `0`               |
| **init_str**    | Modem init string                      | `e1v1q0`          |
| **conn_str**    | Connection message in verbose mode     | `CONNECT 9600`    |
| **conn_code**   | Connection message in non-verbose mode | `1`               |
| **conn_delay**  | Delay before connect message           | `2000`            |
| **ring_delay**  | Delay between rings                    | `2000`            |
| **keep_conn**   | Keep first connection                  | `true`            |
| **wait_at**     | Wait AT before accepting connections   | `false`           |
| **dial_port**   | Default dial port                      | `7740`            |
| **info_val**    | Info strings (ATIn)                    | `N4M-MODEM`       |
| **imsi**        | imsi                                   | `214032081166450` |
| **csq**         | csq                                    | `21,99`           |
| **dtt**         | Dial translation table                 | `[]`              |


### Pubsub topics

| Topic                    | Type   | Description                                           |
| :----------------------- | :----- | :---------------------------------------------------- |
| *evt*                    |        |
| `instance_name`.evt.stat | String | update the state, getting it from the modem instance. |

___

## ACQCARD

### Description
Is responsible for bridging with the data acquisition card (advertisim/gsr)

### Configuration example

    {
        ... other instances ...
        "acqcard": {
            "alerts": "[]",
            "autostart": true,
            "module": "acqcard",
            "pins": {
                "pin00": {
                    "label": "#ARROW_UP",
                    "visibility": true
                },
                "pin01": {
                    "label": "#FLOOR_BIT_1",
                    "visibility": true
                },
                "pin02": {
                    "label": "#FLOOR_BIT_2",
                    "visibility": true
                },
                "pin03": {
                    "label": "#FLOOR_BIT_3",
                    "visibility": true
                },
                "pin04": {
                    "label": "#FLOOR_BIT_4",
                    "visibility": true
                },
                "pin05": {
                    "label": "#FLOOR_BIT_5",
                    "visibility": true
                },
                "pin06": {
                    "label": "#FLOOR_BIT_6",
                    "visibility": true
                },
                "pin07": {
                    "label": "#FLOOR_BIT_7",
                    "visibility": true
                },
                "pin08": {
                    "label": "#FLOOR_BIT_8",
                    "visibility": true
                },
                "pin09": {
                    "label": "#FLOOR_BIT_0",
                    "visibility": true
                },
                "pin10": {
                    "label": "#ARROW_DOWN",
                    "visibility": true
                },
                "pin11": {
                    "label": "#IN_MOTION",
                    "visibility": true
                },
                "pin12": {
                    "label": "#UNDER_MAINTENANCE",
                    "visibility": true
                },
                "pin13": {
                    "label": "#OPEN_DOORS",
                    "visibility": true
                },
                "pullup": {
                    "label": "Pull Up",
                    "visibility": true
                },
                "relay": {
                    "label": "Relay",
                    "visibility": true
                }
            },
            "tty": "/dev/ttyACM0"
        }      
        ... other instances ...
    }

### Specific module parameters

| Parameter                | Description                  | Default Value     |
| :----------------------- | :--------------------------- | :---------------- |
| **telemetry_type**       | telemetry type               | `liftcontroller ` |
| **subelement_telemetry** | item that is doing telemetry | `cabin1`          |
| **tty**                  | TTY device path              | ` `               |
| **baudrate**             | Port baudrate                | `9600`            |
| **alerts**               | JSON describing the alerts   | `[] `             |
| **pins**                 | Pins definition              | ``                |



### Pubsub topics

| Topic                           | Type    | Description                                                |
| :------------------------------ | :------ | :--------------------------------------------------------- |
| *evt*                           |         |
| `instance_name`.evt.pins.*      | Integer | Configurable alerts on pin changes and alerts are managed. |
| *cmd*                           |         |
| `instance_name`.cmd.raw.*       | Buffer  | modify output pins                                         |
| `instance_name`.cmd.relay       | Integer | turn the relay on or off                                   |
| `instance_name`.cmd.get.inputs  | String  | get the value of the inputs                                |
| `instance_name`.cmd.get.outputs | String  | get the value of the outputs                               |

___

## ADC

### Description

Reads the ADC state
### Configuration example

    {
        ... other instances ...
        "adc": {
            "autostart": true,
            "module": "adc",
            "voltage_factor": 0.0085,
            "voltage_path": "/sys/devices/platform/i2c-gpio.0/i2c-0/0-004d/in0_input",
            "voltage_read_secs": 2.5
        }
      ... other instances ...
    }

### Specific module parameters

| Parameter                   | Description                                    | Default Value                                             |
| :-------------------------- | :--------------------------------------------- | :-------------------------------------------------------- |
| **openwrt-cfg_instance**    | TTY type (TTY, PTY, SOCK, ...)                 | `openwrt-cfg `                                            |
| **voltage_path**            | TTY device path                                | `/sys/devices/platform/i2c-gpio.0/i2c-0/0-004d/in0_input` |
| **voltage_read_secs**       | TTY WatchDog timer                             | `2.5`                                                     |
| **feed_missing_alert**      | TTY WatchDog timer                             | `9`                                                       |
| **feed_missing_safe**       | TTY WatchDog timer                             | `18`                                                      |
| **undervolt_alert**         | TTY WatchDog timer                             | `14`                                                      |
| **undervolt_safe**          | TTY WatchDog timer                             | `14.5`                                                    |
| **overvolt_alert**          | Overvolt Alert                                 | `23`                                                      |
| **overvolt_safe**           | Overvolt Safe                                  | `23.5`                                                    |
| **alert_time_feed_missing** | elapsed seconds without feed                   | `900`                                                     |
| **alert_time_undervolt**    | elapsed time being voltage under the low limit | `900`                                                     |
| **alert_time_overvolt**     | elapsed time being voltage over the high limit | `900`                                                     |


### Pubsub topics

| Topic                            | Type    | Description                                                                                |
| :------------------------------- | :------ | :----------------------------------------------------------------------------------------- |
| *evt*                            |         |
| `instance_name`.evt.voltage      | String  | Publish voltage                                                                            |
| sys.alert.`instance_name.type`   | String  | Publish alert, type: feed_missing, overvoltage, undervoltage                               |
| `instance_name`.evt.`alert type` | Integer | Publish 0 for disable alert and 1 to active, type :feed_missing, overvoltage, undervoltage |
| *info*                           |         |
| info.gsr_version                 | String  | Get hw version and edit to voltage_factor                                                  |
___

## ALSAPLAY (Deprecated)

### Description
play sound
### Configuration example

    {
        ... other instances ...
        "aplay": {
            "autostart": true,
            "module": "alsaplay",
            "dev": "plug:hw:0",
            "period_time": 20000,
            "buffer_time": 0
        },
      ... other instances ...
    }

### Specific module parameters

| Parameter       | Description | Default Value     |
| :-------------- | :---------- | :---------------- |
| **alsadev**     | alsa device | `plughw:0,0`      |
| **period_time** | period_time | `20000`           |
| **buffer_time** | buffer time | `period_time *10` |



### Pubsub topics

| Topic                       | Type    | Description            | Flags                                |
| :-------------------------- | :------ | :--------------------- | :----------------------------------- |
| *evt*                       |         |                        |
| `instance_name`.evt.needpcm | Integer | Publish size of buffer | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.pcmsend | Buffer  | Send to buffer         | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |

___

## ALSAREC (Deprecated)

### Description
Play sound

### Configuration example

    {
        ... other instances ...
        "arec": {
            "autostart": true,
            "module": "alsarec",
            "dev": "plug:hw:0",
            "period_time": 20000,
            "buffer_time": 0
            }
      ... other instances ...
    }

### Specific module parameters

| Parameter       | Description | Default Value    |
| :-------------- | :---------- | :--------------- |
| **alsadev**     | alsa device | `plughw:0,0`     |
| **period_time** | period_time | `20000`          |
| **buffer_time** | buffer time | `period_time *4` |



### Pubsub topics

| Topic                   | Type   | Description       | Flags                                |
| :---------------------- | :----- | :---------------- | :----------------------------------- |
| *evt*                   |        |                   |
| `instance_name`.evt.pcm | Buffer | Publish to buffer | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |

___
## ALSAPLAY (Deprecated)

### Description
play sound
### Configuration example

    {
        ... other instances ...
        "aplay": {
            "autostart": true,
            "module": "alsaplay",
            "dev": "plug:hw:0",
            "period_time": 20000,
            "buffer_time": 0
        },
      ... other instances ...
    }

### Specific module parameters

| Parameter       | Description | Default Value     |
| :-------------- | :---------- | :---------------- |
| **alsadev**     | alsa device | `plughw:0,0`      |
| **period_time** | period_time | `20000`           |
| **buffer_time** | buffer time | `period_time *10` |



### Pubsub topics

| Topic                       | Type    | Description            | Flags                                |
| :-------------------------- | :------ | :--------------------- | :----------------------------------- |
| *evt*                       |         |                        |
| `instance_name`.evt.needpcm | Integer | Publish size of buffer | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.pcmsend | Buffer  | Send to buffer         | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |

___

## ALSAREC (Deprecated)

### Description
Play sound

### Configuration example

    {
        ... other instances ...
        "arec": {
            "autostart": true,
            "module": "alsarec",
            "dev": "plug:hw:0",
            "period_time": 20000,
            "buffer_time": 0
            }
      ... other instances ...
    }

### Specific module parameters

| Parameter   | Description | Default Value |
| :---------- | :---------- | :------------ |
| **alsadev** | alsa device | `plughw:0,0`  |
**perio
## ALSAPLAY (Deprecated)

### Description
play sound
### Configuration example

    {
        ... other instances ...
        "aplay": {
            "autostart": true,
            "module": "alsaplay",
            "dev": "plug:hw:0",
            "period_time": 20000,
            "buffer_time": 0
        },
      ... other instances ...
    }

### Specific module parameters

| Parameter       | Description | Default Value     |
| :-------------- | :---------- | :---------------- |
| **alsadev**     | alsa device | `plughw:0,0`      |
| **period_time** | period_time | `20000`           |
| **buffer_time** | buffer time | `period_time *10` |



### Pubsub topics

| Topic                       | Type    | Description            | Flags                                |
| :-------------------------- | :------ | :--------------------- | :----------------------------------- |
| *evt*                       |         |                        |
| `instance_name`.evt.needpcm | Integer | Publish size of buffer | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.pcmsend | Buffer  | Send to buffer         | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |

___

## AMBER (Deprecated)

### Description
mod_amber is used to receive frames from meters compatible with wM-BUS.

    http://wiki.nayarsystems.com/display/H72/Modulo+Amber+Wireless+wM-BUS
### Configuration example

    {
        ... other instances ...
        "amber" : {
            "module"            : "amber",
            "respawn"           : true,
            "max_respawn_delay" : 60.0,
            "autostart"         : true,
            "log_level"         : "debug",
            "baudrate"          : 9600,
            "tty_wmbus"         : "/dev/ttyUSB0",
            "frame_rssi"        : true,
            "wmbus_mode"        : 8,
            "max_packet_len"    : 250,
            "encryption"        : false
        }
      ... other instances ...
    }

### Specific module parameters

| Parameter          | Description        | Default Value  |
| :----------------- | :----------------- | :------------- |
| **tty/tty_wmbus**  | Serial TTY device  | `/dev/ttyUSB0` |
| **baudrate**       | Serial baudrate.   | `9600`         |
| **frame_rssi**     | buffer time        | `false`        |
| **wmbus_mode**     | operation mode     | `3`            |
| **max_packet_len** | max packet len     | `250`          |
| **encryption**     | enable encryption? | `false`        |


### Pubsub topics
| Topic                        | Type    | Description                                      | Flags                                |
| :--------------------------- | :------ | :----------------------------------------------- | :----------------------------------- |
| *evt*                        |         |                                                  |
| `instance_name`.evt.stat     | Integer | Publish stat                                     |
| `instance_name`.evt.bstat    | Integer | Publish amber stat                               |
| `instance_name`.evt.recv     | Buffer  | Publish command frame received from amber device | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.recv.raw | Buffer  | Publish raw                                      | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `instance_name`.evt.json     | Buffer  | Publish  json frame                              |
| `instance_name`.evt.timeout  | Integer | Publish  timeout                                 | MSG_FL_INSTANT                       |

___


## BCAST

### Description
module that is responsible for making a retransmission of the topics that have been assigned.

### Configuration example

    {
        ... other instances ...
        "bcast": {
            "module": "bcast",
            "interface_name": "enp2s0",
            "topics": [
                "console.evt.mod.stat",
                "test"
            ]
        }
      ... other instances ...
    }

### Specific module parameters

| Parameter             | Description       | Default Value  |
| :-------------------- | :---------------- | :------------- |
| **broadcast_port**    | broadcast port    | `3334`         |
| **broadcast_address** | broadcast address | `239.255.42.1` |
| **wdinet_instance**   | wdinet instance   | `wdinet`       |
| **interface_name**    | interface name    | `eth0`         |
| **topics**            | Topics list       | `[ ]`          |


### Pubsub topics
| Topic                                              | Type   | Description                                                                            | Flags                                |
| :------------------------------------------------- | :----- | :------------------------------------------------------------------------------------- | :----------------------------------- |
| *evt*                                              |        |                                                                                        |
| `instance_name`.evt.msgin.`topic`                  | Buffer | Publish buffer                                                                         |
| `instance_name_wdinet`.nl.newlink.`interface_name` | Buffer | Publish command frame received from amber device                                       | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| symsys.stat.device                                 | Buffer | Publish buffer                                                                         | MSG_FL_STICKY / MSG_ENC_MSGPACK      |
| subscribe - symsys.stat.device                     | *      | Depending on the data received, one thing or another is done. The message is processed |
| subscribe - `topic list`                           | *      | Depending on the data received, one thing or another is done. The message is processed |
| *cmd*                                              |        |                                                                                        |
| `instance_name`.cmd.query                          | String | receives a json and stores the necessary information product, id, topic                |


___


## DIALER

### Description
 Generic dialer to enable the possibility of using several types of dialers (mod_sip, mod_modem) at the same time in other modules such as "mod_gw", "mod_telealarm" or "mod_tas
### Configuration example

    {
        ... other instances ...
        "dialer": {
            "autostart": true,
            "dialer_instance_0": "modem",
            "dialer_instance_1": "sip",
            "dialer_instance_2": "",
            "dialer_instance_3": "",
            "dialer_instance_4": "",
            "log_level": "debug",
            "module": "dialer",
            "respawn": true
        },
      ... other instances ...
    }

### Specific module parameters

| Parameter             | Description                                            | Default Value |
| :-------------------- | :----------------------------------------------------- | :------------ |
| **dialer_instance_0** | dialer instance                                        | ` `           |
| **dialer_instance_1** | dialer instance                                        | ` `           |
| **dialer_instance_2** | dialer instance                                        | ` `           |
| **dialer_instance_3** | dialer instance                                        | ` `           |
| **fallback**          | activate to publish the *evt.linkstatus* of the dialer | `false`       |


### Pubsub topics
| Topic                              | Type   | Description                                                                                    | Flags |
| :--------------------------------- | :----- | :--------------------------------------------------------------------------------------------- | :---- |
| *evt*                              |        |                                                                                                |
| `instance_name`.evt.stat           | String | Activate to flags evt                                                                          |
| `instance_name_sip`.evt.linkstatus | String | publish the *evt.linkstatus* of the dialer, if fallbak is true                                 |
| *cmd*                              |        |                                                                                                |
| `instance_name`.cmd.hangup         | String | hangup to call                                                                                 |
| `instance_name`.cmd.answer         | String | answet to call                                                                                 |
| `instance_name`.cmd.qbstat         | String | Publish dialer stat to `evt.linkstatus`, stats (unsetm setup, idle,dial, ring,online, failure) |
| `instance_name`.cmd.dial           | String | Make call                                                                                      |
| `instance_name`.cmd.dial_result    | String | Return call result (error,ok), if result is error , try call                                   |
| `instance_name`.cmd.dtmfsend       | String | Publish dtmf                                                                                   |
| `instance_name`.cmd.pcmsend        | String | Publish pcm buffer                                                                             |

___


## EDEL (Deprecated)

### Description
Module to communicate with edel maneuvers
### Configuration example

    {
        ... other instances ...
        "edel": {
            "autostart": true,
            "serial_instance": "serial",
            "module": "edel"

        },
      ... other instances ...
    }

### Specific module parameters

| Parameter           | Description     | Default Value |
| :------------------ | :-------------- | :------------ |
| **serial_instance** | serial instance | ` serial`     |



### Pubsub topics
| Topic                       | Type   | Description                                                                                                                                       | Flags |
| :-------------------------- | :----- | :------------------------------------------------------------------------------------------------------------------------------------------------ | :---- |
| `instance_name_serial`.recv | Buffer | rreceives a buffer, analyzes the information and publishes a json in sys.alert.`instance_name` with the information received plant and error code |

___

## GB_BRIDGE 

### Description
Module in charge of transforming real time data and sending them by mqtt.
### Configuration example

    {
        ... other instances ...
        "gb_bridge": {
            "autostart": true,
            "module": "gb_bridge",
            "smart_instance": "smart@123",
            "mqtt_instance": "mqtt",
            "tz": "CST-8",
            "refresh_period": 600
	    },
      ... other instances ...
    }

### Specific module parameters

| Parameter          | Description     | Default Value |
| :----------------- | :-------------- | :------------ |
| **smart_instance** | serial instance | ` `           |
| **mqtt_instance**  | serial instance | ` `           |
| **refresh_period** | refresh period  | `600`         |
| **tz**             | tz              | `CST-8`       |


### Pubsub topics
| Topic                                                | Type    | Description                                                                             | Flags |
| :--------------------------------------------------- | :------ | :-------------------------------------------------------------------------------------- | :---- |
| *evt*                                                |         |                                                                                         |
| `instance_name_mqtt`.evt.mod.stat                    | Integer | a 0 is published in *`instance_name_smart`.qstat* if the status is 1                    |
| `instance_name_smart`.evt.telemetry.liftcontroller.* | Buffer  | received event, the information is treated and published in `instance_name_mqtt`cmd.pub |
| `instance_name`.status.refresh.`instance_name_smart` | String  | a json is received and the cabin status is sent                                         |
| *cmd*                                                |         |                                                                                         |
| `instance_name_mqtt`cmd.pub                          | String  | Publish Json                                                                            |
___


## GPIO

### Description
Module in charge of transforming real time data and sending them by mqtt.
### Configuration example

    {
        ... other instances ...
        "gpio": {
            "autostart": true,
            "module": "gpio",
            "pin": "12",
            "direction": "in",
            "initial_value": "0",
            "debounce": 0.01
	    },
      ... other instances ...
    }

### Specific module parameters

| Parameter         | Description              | Default Value |
| :---------------- | :----------------------- | :------------ |
| **pin**           | pin id                   | ` -1`         |
| **direction**     | pin direcction in or out | ` in`         |
| **initial_value** | default value            | `0`           |
| **debounce**      | time of repeat           | `0.01`        |


### Pubsub topics
| Topic                              | Type    | Description                                            | Flags          |
| :--------------------------------- | :------ | :----------------------------------------------------- | :------------- |
| *evt*                              |         |                                                        |
| `instance_name`.evt.info           | Buffer  | publish json with value & prev_width                   | MSG_FL_INSTANT |
| *cmd*                              |         |                                                        |
| `instance_name_smart`.cmd.getvalue | Buffer  | publish value                                          |
| `instance_name_mqtt`.cmd.setvalue  | Integer | the pin value is modified, if the pin direction is out |

___


## GSR_LOCALCFG

### Description
module in charge of loading the local configuration of the gsr
### Configuration example

    {
        ... other instances ...
        "gsr-localcfg":	{
            "autostart":	true,
            "log_level":	"none",
            "max_respawn_delay":	60,
            "module":	"gsr-localcfg",
            "respawn":	true
        }
      ... other instances ...
    }

### Specific module parameters

| Parameter             | Description       | Default Value |
| :-------------------- | :---------------- | :------------ |
| **max_respawn_delay** | max_respawn_delay | ` 60`         |

### Pubsub topics
| Topic                                     | Type    | Description                                                                                                                                             | Flags          |
| :---------------------------------------- | :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------- |
| `topic`                                   | All     | Clone msg                                                                                                                                               |
| *cmd*                                     |         |                                                                                                                                                         |
| `instance_name`.cmd.setwm                 | Buffer  | publish value                                                                                                                                           |
| `instance_name`.cmd.settap100             | String  | modify the configuration of pq00 , editing the id of the card_number                                                                                    |
| `instance_name`.cmd.setblock              | Integer | edit config, if value is 1  turn off modules (sms, modem, button_config,button_factory) else turn on modules (sms, modem, button_config,button_factory) |
| sys.cmd.cfg.set.`instance_name.item_name` | String  | Publish json                                                                                                                                            | MSG_FL_INSTANT |
| sys.cmd.reload                            | Integer | Ask for the config again on system reloads                                                                                                              |
___


## MQTT

### Description
MQTT client module acting as a bridge between internal pubsub and MQTT protocol (external).

### Configuration example

    {
        ... other instances ...
        "mqtt": {
            "module": "mqtt",
            "max_respawn_delay": 10,
            "autostart": true,
            "log_level": "debug",
            "respawn": true,
            "server_address": "localhost",
            "server_port": 1883,
            "username": "example",
            "password": "1234",
            "ssl": false,
            "keep_alive": 30,
            "clean_session": true,
            "last_will_topic": "bye",
            "last_will_message": "bye",
            "last_will_qos": 0,
            "last_will_retain": true,
            "sub_internal": [
                {
                    "topic": "max17048.evt.*",
                    "qos": 0
                },
                {
                    "topic": "modem.evt.*",
                    "qos": 2
                }
            ],
            "sub_external": [
                {
                    "topic": "obelisk/$id/rpc/+/max17048/evt/#",
                    "qos": 0
                },
                {
                    "topic": "obelisk/$id/rpc/+/modem/evt/#",
                    "qos": 2
                }
            ]
        }
      ... other instances ...
    }

### Specific module parameters

| Parameter            | Description       | Default Value     |
| :------------------- | :---------------- | :---------------- |
| **server_address**   | server address    | ` `               |
| **server_port**      | server port       | ` 1883`           |
| **user_name**        | user              | ` `               |
| **password**         | password          | ` `               |
| **keapalive**        | time of keapalive | ` 30`             |
| **ssl**              | ssl               | ` false`          |
| **clean_session**    | clean session     | ` true`           |
| **last_will_topic**  | last_will_topic   | `obelisk/$id/bye` |
| **last_will_mesage** | last_will_mesage  | `connection lost` |
| **last_will_qos**    | last_will_qos     | `0`               |
| **last_will_retain** | last_will_retain  | `false`           |



### Pubsub topics
| Topic                             | Type    | Description                | Flags                         |
| :-------------------------------- | :------ | :------------------------- | :---------------------------- |
| *cmd*                             |         |                            |
| `instance_name`.cmd.internal.sub  | String  | Subscribe internal topic   |
| `instance_name`.cmd.external.sub  | String  | Subscribe external topic   |
| `instance_name`.cmd.internal.usub | Buffer  | Unsubscribe internal topic |
| `instance_name`.cmd.external.usub | String  | Unsubscribe external topic |
| `instance_name`.cmd.pub           | Integer | String                     | recive Json & subscribe topic |
| `instance_name`.evt.rpc.*         | String  | Send mesage to mqtt        |

____



## TAS

### Description
Module that implements a P100 telealarm server (TAS)
### Configuration example

    {
        ... other instances ...
       	"tas": {
            "autostart": true,
            "module": "tas",
            "slic_instance": "slic",
            "slic_pcm_gain": -5,
            "modem_pcm_gain": 2,
            "modem_instance": "modem",
            "gw_instance": "gateway",
            "protocol": "p100",
            "p100_start_delay": 2,
            "p100_pkt_input_timeout": 2,
            "p100_disconnect_delay": 2
        }
      ... other instances ...
    }

### Specific module parameters

| Parameter          | Description  | Default Value |
| :----------------- | :----------- | :------------ |
| **slic_pcm_gain**  | slic pcm     | ` 0`          |
| **modem_pcm_gain** | pcm again    | ` 0`          |
| **protocol**       | protocol use | ` p100`       |




### Pubsub topics
| Topic                                 | Type    | Description                                   | Flags                    |
| :------------------------------------ | :------ | :-------------------------------------------- | :----------------------- |
| *evt*                                 |         |                                               |
| `instance_name`.evt.stat              | String  | Active evt.stat                               |
| `instance_name`.evt.instance.ended    | String  | Instance ended                                |
| `instance_name`.evt.slic_dtmfsend     | String  | If buffer_used is more than 0, send to buffer |
| `instance_name`.evt.modem_dtmfsend    | String  | Send buffer                                   |
| `instance_name_gw`.evt.stat           | String  | update gw_stat                                |
| `instance_name_slic`.evt.dtmf         | String  |
| `instance_name_slic`.evt.onhook       | String  | on hook call                                  |
| `instance_name_slic`.evt.pcm          | String  | Publish pcm buffer in slic.cmd.pcmsend        | MSG_FL_NONRECURSIVE      |
| `instance_name_modem`.evt.pcm         | String  | Publish pcm buffer in modem.cmd.pcmsend       | MSG_FL_NONRECURSIVE      |
| `instance_name_modem`.evt.dtmf        | String  |
| `instance_name_modem`.evt.bstat       | String  | Subscribe internal topic                      |
| *cmd*                                 |         |                                               |
| `instance_name`.cmd.setparams         | String  | Set config                                    |
| `instance_name`.cmd.dial              | String  | Make call                                     |
| `instance_name`.cmd.slic_dtmfsend     | Buffer  | Send buffer                                   |
| `instance_name`.cmd.modem_dtmfsend    | String  | forward the message to modem.cmd.dtmfsend     |
| `instance_name`.cmd.disable_pcm_regen | Integer | String                                        | active disable_pcm_regen |
| `instance_name`.evt.enable_pcm_regen  | String  | disable disable_pcm_regen                     |

____



## TCP

### Description
Manages TCP connection
### Configuration example

    {
        ... other instances ...
        "tcp": {
            "module": "tcp",
            "autostart": true,
            "port": 52101,
            "ip_server": "127.0.0.1"
        }
      ... other instances ...
    }

### Specific module parameters

| Parameter           | Description                             | Default Value |
| :------------------ | :-------------------------------------- | :------------ |
| **port**            | port for socket connection              | ` 52101`      |
| **ip_server**       | Ip for socket connection                | ` 127.0.0.1`  |
| **size_buffer**     | Size of the frames that we will receive | ` 8192`       |
| **write_topic**     | Topic to write frames                   | ` p100`       |
| **read_topic**      | Topic to read frames                    | ` p100`       |
| **bridge**          | Activate tcp-bridge mode                | ` false`      |
| **accept_new_conn** | Accept new tcp conection                | ` false`      |
| **timeout**         | TCP connection timeout                  | ` 600`        |
| **whitelist**       | Subnet list permit                      | ` 127.0.0.1`  |



### Pubsub topics
| Topic                     | Type   | Description                        | Flags                                |
| :------------------------ | :----- | :--------------------------------- | :----------------------------------- |
| *evt*                     |        |                                    |
| `instance_name`.evt.recv  | Buffer | Publish the buffer received by tcp | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| `write_topic`             | Buffer | Publish the buffer received by tcp | MSG_FL_INSTANT / MSG_FL_NONRECURSIVE |
| *cmd*                     |        |                                    |
| `instance_name`.cmd.write | Buffer | Send the buffer by tcp             |
| `read_topic`              | Buffer | Send the buffer by tcp             |
____


## EPCL

### Description
EPCL is responsible for measuring the kinetic energy recovery system
### Configuration example

    {
        ... other instances ...
        "epcl": {
            "autostart": true,
            "module": "epcl",
            "log_level": "debug",
            "can_instance": "socketcan",
            "epcl_id": 0 
        }
        ... other instances ...
    }

### Specific module parameters

| Parameter                    | Description                                             | Default Value |
| :--------------------------- | :------------------------------------------------------ | :------------ |
| **hist_battery_voltage**     | battery voltage threshold publish value (0.1V/1)        | ` 5`          |
| **hist_battery_current**     | battery current threshold publish value (1A/1)          | `5`           |
| **hist_dc_power**            | DC link power threshold publish value (1W/1)            | ` 20`         |
| **hist_dc_voltage**          | DC link current threshold publish value (0.1A/1)        | ` 30`         |
| **hist_dc_current**          | Topic to read frames                                    | ` 5`          |
| **hist_ac_charger_current**  | AC charger current threshold publish value (0.1A/1).    | ` 5`          |
| **hist_ac_inverter_current** | AC inverter current threshold publish value (0.1A/1).   | ` 5`          |
| **hist_solar_current**       | Solar charger current threshold publish value (0.1A/1). | ` 5`          |
| **epcl_id**                  | epcl id (0...15)                                        | `0`           |



### Pubsub topics
| Topic                                          | Type    | Description                                                                                 | Flags |
| :--------------------------------------------- | :------ | :------------------------------------------------------------------------------------------ | :---- |
| *evt*                                          |         |                                                                                             |
| `instance_name_can`.evt.frame                  | Buffer  | can frames are received and processed                                                       |
| `instance_name`.evt.telemetry                  | String  | Publish json with status                                                                    |
| `instance_name`.evt.telemetry.dev`epcl_id`     | String  | Publish information only the changed values.                                                |
| sys.evt.telemetry.`instance_name`.dev`epcl_id` | String  | Publish information only the changed values.                                                |
| epcl.telemetry.dev`epcl_id`                    | String  | Publish information only the changed values.                                                |
| *cmd*                                          |         |                                                                                             |
| `instance_name`.cmd.control                    | Buffer  | can frames are sent to the maneuver                                                         |
| `instance_name`.cmd.telemtery.activate         | Integer | if value is 1 telemetry activate, else telemtry disable                                     |
| `instance_name`.cmd.info                       | Buffer  | information is requested through a publication can.cmd.frame                                |
| `instance_name`.cmd.telemtery.status           | Buffer  | publish `instance_name`.cmd.info with value 0, and `instance_name`.cmd.control with value 1 |
____


## NAM

### Description
EPCL is responsible for measuring the kinetic energy recovery system
### Configuration example

    {
        ... other instances ...
        "nam1": {
                "module": "nam",
                "car_number": 1,
                "bcast_instance": "bcast",
                "filter_audio_path": "./audio/estableciendo.wav",
                "ring_audio_path": "./audio/ringing.wav"
            },
        ... other instances ...
    }

### Specific module parameters

| Parameter                       | Description                                  | Default Value |
| :------------------------------ | :------------------------------------------- | :------------ |
| **filter_audio_path**           | audio file path                              | ` `           |
| **filter_countdown_audio_path** | audio file path                              | ` `           |
| **ring_audio_path**             | Ring audio file path                         | ` `           |
| **car_number**                  | Cabin number to use                          | ` -1`         |
| **soc_alert**                   | Percentage to trigger the low battery alert  | ` 25`         |
| **soc_alert_error**             | Percentage to trigger the battery error aler | ` 10`         |
| **max_vbat**                    | Maximum battery voltage in mV                | ` 4100`       |
| **min_vbat**                    | Minimum battery voltage in mV                | ` 3200`       |
| **mic_volume**                  | Microphone volume                            | ` 5`          |
| **speaker_volume**              | Cabin speaker volume                         | ` 5`          |
| **leds_mode_minimum**           | Configure minimal mode for LEDs              | ` false`      |
| **daytime_speech_volume**       | Voice synthesis volume during daytime        | ` 5`          |
| **daytime_hour_start**          | Daytime start time in 24h format             | ` 6`          |
| **daytime_hour_end**            | Daytime start time in 24h format             | ` 22`         |



### Pubsub topics
| Topic                                                     | Type    | Description                                                                            | Flags         |
| :-------------------------------------------------------- | :------ | :------------------------------------------------------------------------------------- | :------------ |
| *evt*                                                     |         |                                                                                        |
| `instance_name_wdinet`.evt.nl.newlink.`interface_name`    | String  | stop module                                                                            |
| `SYMSYS_NAM_STAT_TOPIC`.evt.msgin.`instance_name_bcast`   | String  | the module is configured, with the information of the physical audio module            |
| `SYMSYS_POWER_STAT_TOPIC`.evt.msgin.`instance_name_bcast` | String  | voltage information is obtained and alerts are verified                                |
| `instance_name_audiogen`.evt.started                      | Buffer  | audio playing is set to true                                                           |
| `instance_name_audiogen`.evt.stopped                      | Buffer  | audio playing is set to false, and active alarm_status                                 |
| `nam.evt.state`                                           | String  | Publish  json with information, {index, call, alarm, test,mic_volum,speaker_volume}    | MSG_FL_STICKY |
| `instance_name`.evt.battery.stat                          | Integer | Publish state battery {BAT_LOW: 1,BAT_LOW_CLEAR: 2, BAT_ERROR: 3, BAT_ERROR_CLEAR: 4 } |
| *cmd*                                                     |         |                                                                                        |
| `instance_name`.cmd.pcmsend                               | Buffer  | Publish send to buffer.                                                                |
| `instance_name`.cmd.call_status                           | String  | update call status ,set CALL_STARTED                                                   |
| `instance_name`.cmd.failed_alarm                          | String  | update call status ,set CALL_ENDED                                                     |
| `instance_name`.cmd.end_of_alarm                          | String  | update call status ,set FALSE                                                          |
| `instance_name`.cmd.eoa                                   | String  | update call status ,set FALSE                                                          |
| `instance_name`.cmd.interphone.start                      | String  | update call status ,set CALL_SUCCEDED                                                  |
| `instance_name`.cmd.interphone.stop                       | String  | update call status ,set CALL_ENDED                                                     |
| `instance_name`.cmd.set_test_result                       | String  | update test status                                                                     |
| `instance_name`.cmd.tx                                    | String  | packages are sent                                                                      |

____

## MK_BRIDGE

### Description
Bridge between mk and server
### Configuration example

    {
        ... other instances ...
       "mk_bridge": {
            "log_level": "debug",
            "module": "mk_bridge",
            "autostart": true,
            "serial_instance": "serial"
        },  
        ... other instances ...
    }

### Specific module parameters

| Parameter           | Description     | Default Value |
| :------------------ | :-------------- | :------------ |
| **ip_server**       | ip server       | ` `           |
| **port**            | port server     | ` `           |
| **serial_instance** | serial instance | `serial`      |
| **modem_instance**  | modem instance  | ` modem`      |




### Pubsub topics
| Topic                              | Type    | Description                                              | Flags |
| :--------------------------------- | :------ | :------------------------------------------------------- | :---- |
| *evt*                              |         |                                                          |
| `instance_name`.evt.stat           | Integer | update stat run machine                                  |
| `instance_name`.evt.recv.imei      | String  | Internal publication of the modem status to obtain IMEI. |
| `instance_name_modem`.state        | String  | Requesting modem status                                  |
| `instance_name_modem`.evt.mod.stat | String  | Request modem stat                                       |
| `instance_name_serial`.evt.recv    | String  | Receive what the serial publishes                        |
| *cmd*                              |         |                                                          |
| `instance_name_serial`.cmd.write   | String  | It is published what will be written by the serial       |


____

## NEXUS

### Description
Maintains a connection with nexus and listens for commands
### Configuration example

    {
        ... other instances ...
        "nexus": {
            "module": "nexus",
            "proto":"tcp",
            "server":"localhost",
            "port": 1717,
            "user": "test",
            "pass": "test",
            "watchdog": 95,
            "keepalive": 45,
            "max_respawn_delay": 3,
            "pullpath": "test.gsr.pubsub"
        },
        ... other instances ...
    }

### Specific module parameters

| Parameter       | Description                                                                                        | Default Value      |
| :-------------- | :------------------------------------------------------------------------------------------------- | :----------------- |
| **proto**       | Connection protocol                                                                                | `ssl`              |
| **server_addr** | Server address                                                                                     | `127.0.0.1`        |
| **user**        | User                                                                                               | `root`             |
| **pass**        | Password                                                                                           | ` root`            |
| **pullpath**    | Path where the module will do a pull, offering a service of pipe exchange for access to the pubsub | ` test.gsr.pubsub` |
| **server_port** | Server port (deprecated)  ` 1717`                                                                  |
| **watchdog**    | Time after which the connection is closed if no information is received from the server            | ` 95`              |

### Pubsub topics
| Topic                                    | Type   | Description                                           |
| :--------------------------------------- | :----- | :---------------------------------------------------- |
| *evt*                                    |        |                                                       |
| `instance.REQUEST_PATH`                  | str    | Publish request                                       |
| `instance_name`.response.`child->teskid` | str    | publish a json with the corresponding information     |
| `instance_name`.response.firstlogin      | str    | if login is ok, unsubscribe to path, else stop module |
| `instance_name`.response.pipes           | str    | publish a json with the corresponding information     |
| *cmd*                                    |        |                                                       |
| `instance_name`.cmd.request              | String | Reply to the post                                     |
____


## GSR_LOCALCFG

### Description
obelisk configuration control module
### Configuration example

    {
        ... other instances ...
        "gsr-localcfg":	{
            "autostart":	true,
            "log_level":	"none",
            "max_respawn_delay":	60,
            "module":	"gsr-localcfg",
            "respawn":	true
        },
        ... other instances ...
    }

### Specific module parameters

| Parameter | Description | Default Value |
| :-------- | :---------- | :------------ |
| **-**     | -           | `-`           |


### Pubsub topics
| Topic                        | Type   | Description                                                                  |
| :--------------------------- | :----- | :--------------------------------------------------------------------------- |
| *cmd*                        |        |                                                                              |
| `instance_name`.cmd.swtwm    | int    | Change working mode. 0 -> Gateway, other -> telealarm and msg->int is car id |
| `instance_name`.cmd.settp100 | String | Set new p100 id                                                              |
| `instance_name`.cmd.setblock | int    | if value is one, block : sms, modem, buttons, else active sms, modem,buttons |
| sys.cmd.cfg.save             | int    | seva config                                                                  |
| sys.cmd.reload               | int    | reload obelisk                                                               |
| sys.cmd.get.`instance`       | int    | Get config to instance                                                       |
| sys.cmd.set.`instance.item`  | str    | Set config to instance                                                       |
| sys.cmd.cfg.set.`instance`   | str    | Set new config to instance                                                   |
____


## IPBUS

### Description
Create layer 2 bridge and implements ipbus protocolo
### Configuration example

    {
        ... other instances ...
            "ipbus":{
                "autostart": true,
                "module": "ipbus",
                "uart_dev": "/dev/ttyUSB0",
                "tap_dev": "ipbus0",
                "uart_baudrate": 1000000,
                "ipbus_loop_period": 0.003
            }
        ... other instances ...
    }

### Specific module parameters

| Parameter                           | Description                                        | Default Value |
| :---------------------------------- | :------------------------------------------------- | :------------ |
| **uart_dev**                        | tty path to use comunication board                 | ` `           |
| **tap_dev**                         | Name of the network interface that will be created | `ipbus0`      |
| **ipbus_proto_hdm_baudrate**        | HDM baudrate                                       | ` 0`          |
| **ipbus_proto_hdm_preambles**       | Number of preambles                                | ` 3`          |
| **ipbus_proto_hdm_max_tx_burst**    |                                                    | ` 9084`       |
| **ipbus_proto_hdm_inactive_bus_us** |                                                    | ` 5000`       |
| **ipbus_read_buffer_len**           | Entry buffer size                                  | ` 1024*4`     |
| **ipbus_max_size**                  | Max size ipbus frame                               | ` 1550`       |
| **ipbus_max_tx_queue_len**          | Max size to send frame queue                       | ` 40`         |
| **ipbus_loop_period**               | Period in ms of the ipbus main loop                | ` 0.005`      |

### Pubsub topics
| Topic                         | Type | Description                                                                  |
| :---------------------------- | :--- | :--------------------------------------------------------------------------- |
| *evt*                         |      |                                                                              |
| `instance_name`.evt.stat      | int  | change stat and call run_machine                                             |
| `instance_name`.evt.bstat     | int  | change bstat                                                                 |
| `instance_name`.evt.th_result | int  | activate event.timeout, set th->result whith msg->int , and call run_machine |
| `instance_name`.evt.timeout   | int  | activate event.timeout, and call run_machine                                 |

____

## NANO

### Description
Leitronik nano audio module compatibility
### Configuration example

    {
        ... other instances ...
    "nano":         {
        "autostart":    	true,
        "module":       	"nano",
        "modem":        	"modem",
        "slic":         	"slic",
        "car_number":   	1,
        "pin":				"12345",
        "required_pin": 	true,
        "pin_max_retry":	3,
        "call_max_time":	900,
        "press_button_time": 3
    },
        ... other instances ...
    }

### Specific module parameters

| Parameter             | Description                                | Default Value |
| :-------------------- | :----------------------------------------- | :------------ |
| **modem_instance**    | Modem instance name                        | ` `           |
| **slic_instance**     | slic instance name                         | ` `           |
| **press_button_time** | Time in seconds to trigger emergency call  | `3`           |
| **pin**               | Incoming call pin, if its required         | `12345`       |
| **required_pin**      | Activate pin required for incomming calls  | `false`       |
| **call_max_time**     | Max call time duration in seconds          | `900`         |
| **pin_max_retry**     | Number of retries if pin fails             | `1`           |
| **auto_test**         | Auto test mode, sends dtmf to disable leds | `false`       |


### Pubsub topics
| Topic                               | Type | Description                                                                           |
| :---------------------------------- | :--- | :------------------------------------------------------------------------------------ |
| *evt*                               |      |                                                                                       |
| `instance_name`.evt.stat            | int  | change stat and call run_machine                                                      |
| `instance_name_modem`.evt.dtmf      | str  | revived dtmf of module_instance                                                       |
| `instance_name_slic`.evt.mod.stat   | int  | recive stat of slic_instance                                                          |
| `instance_name_slic`.evt.dtmf       | str  | revived dtmf of instance_instance                                                     |
| `instance_name_slic`.evt.pcm        | str  | revived pcm of instance_instance and publish pcm in  `instance_name`.evt.pcm          |
| *cmd*                               |      |                                                                                       |
| `instance_name`.cmd.pcmsend         | str  | revived pcm of instance_instance and publish pcm in  `instance_name_slic`.cmd.pcmsend |
| `instance_name`.cmd.interphone.*    | int  | revived interphone state and publish MN_EV_INCOMING_CALL or  MN_EV_STOP_CALL          |
| `instance_name`.cmd.failed_alarm    | str  | Publish `fail` in rtopic                                                              |
| `instance_name`.cmd.end_of_alarm    | str  | Publish `end` in rtopic                                                               |
| `instance_name`.cmd.call_status     | int  | Set status_ev `1 MN_EV_START_CALL                                                     | 2 MN_EV_ALARM_VALIDATE | 3 MN_EV_STOP_CALL` |
| `instance_name`.cmd.emerg           | str  | Set rtopic and set ev ->  MN_EV_EMERGENCY_READY                                       |
| `dtmd_rtopic`                       | str  | Set ev ->  MN_EV_DTMF_CB                                                              |
| `instance_name`.cmd.autotest        | str  | Set rtopic and set ev ->  MN_EV_TEST                                                  |
| `instance_name`.cmd.set_test_result | str  | If msg->int_val is 1  publish `C` in `instance_name_slic`.cmd.dtmfsend                |
____
