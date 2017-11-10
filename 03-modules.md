# Modules

There are several modules available to Obelisk to instance, the next subsections will explain some of the most important ones.

## Socketcan

### Description

This module is in charge of configuring and managing a CAN network interface as defined in the [Linux Socketcan standard](https://www.kernel.org/doc/Documentation/networking/can.txt). The other modules receive and send CAN frames through this module.

### Configuration example

    {
        ... other instances ...
        "socketcan": {
            "module":	"socketcan",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "bitrate":	200000,
            "interface_name":	"can0",
            "listen_only":	false,
            "loopback":	false,
            "restart_ms":	10,
            "tx_buffer_frame_num":	128
        },
        ... other instances ...
    }

### Specific module parameters

Parameter   | Description   | Default Value 
----------  | -----------   | ------------- 
**interface_name**     | Name of the CAN network interface. | `"can0"`
**bitrate**            | Required field. Bitrate of the bus to which the interface is connected. | `-1`
**listen_only**        | Activate listening mode in which messages can be received but not sent. It also does not intervene on the bus to make the ACK of the messages. | `"true"`
**loopback**           | Activate test mode in which outgoing traffic is redirected as input without going through the bus. | `"false"`
**restart_ms**         | Time in milliseconds to restart the interface automatically in case of error. | `10`
**tx_buffer_frame_num**| Size of the frame send buffer. | `128`

### Pubsub topics

Topic                     | Type |   Description  |
----------                | ---- |   -----------  |
`instance_name`.evt.frame | Byte Array | Subscribe to enable the reception of CAN frames from the CANBus. |
`instance_name`.evt.error | Byte Array | Subscribe to enable the reception of CAN [error](https://github.com/torvalds/linux/blob/master/include/uapi/linux/can/error.h) frames from the CANBus. |
`instance_name`.cmd.frame | Byte Array | Publish to this topic to send a CAN frame to the CANBus. |

___

## Serial
### Description

This module is in charge for configuring and managing the information through a serial port.

### Configuration example

    }
        ... other instances ...
        "serial":{
            "module":	"serial",
            "log_level":	"none",
            "autostart":	false,
            "respawn":	true,
            "max_respawn_delay":	60,
            "baudrate":	9600,
            "databits":	8,
            "hwfc":	false,
            "parity":	0,
            "read_threshold_time":	0.05,
            "stopbits":	1,
            "swfc":	false,
            "tty":	"/dev/ttySERIAL"
        },
        ... other instances ...
    }

### Specific module parameters

Parameter   | Description   | Default Value 
----------  | -----------   | ------------- 
**tty** | TTY device route. | `"/dev/ttyUSB0"`
**baudrate** | Serial baudrate. | `9600`
**databits** | Data bits of the serial. | `8`
**read_threshold_time** | Delay that is given to the serial to read blocks, instead of sending data byte to byte. | `0.05`
**stopbits** | Stop serial bits. | `1`
**parity** | Parity del serial. | `0`
**hwfc** | Hardware flowcontrol. | `"false"`
**swfc** | Software flowcontrol. | `"false"`

### Pubsub topics

Topic                     | Type |   Description  |
----------                | ---- |   -----------  |
`instance_name`.recv      | Byte Array | Subscribe to enable the reception of data frames from the serial bus. |
`instance_name`.cmd.write | Byte Array | Publish to send data frames to the serial bus. |

___


## Console
### Description

This module is responsible for opening a console that allows the user to hook up to the obbus pubsub to perform debug. It is possible to access the console through a client socket (netcat). The connection data to the console through a socket are defined in the configuration of the module, as shown in the following example.

Once connected to the socket, the available commands are the following:

Command                   | Value Type |   Description  |
----------                | ----    |   -----------  |
ps:`topic`:`value` | String  |  Publish a string message to `topic`. |
pi:`topic`:`value` | Integer |  Publish a integer message to `topic`. |
pf:`topic`:`value` | Float   |  Publish a float message to `topic`.|
s:`topic`  | (ignored)   |  Subscribe to the messages of `topic`. |
u:`topic`  | (ignored)   |  Unsubscribe from the messages of `topic`. |
ua | (ignored)   | Unsubscribe from the messages of all the `topic`s. |

### Configuration example

    }
        ... other instances ...
        "console":{
            "module":	"console",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "max_remote_conns":	4,
            "port":	2323,
            "server_addr":	"0.0.0.0",
            "timeout":	600,
            "blacklist":	"^$",
            "whitelist":	"^(10\\.|127\\.)"
        },
        ... other instances ...
    }

### Specific module parameters

Parameter   | Description   | Default Value 
----------  | -----------   | ------------- 
**server_addr** | IP address of the server. | `"127.0.0.1"`
**port** | Server port. | `2323`
**max_remote_conns** | Maximum number of remote connections. | `4`
**timeout** | Seconds of inactivity before closing the remote connection to prevent it from being permanently open. It does not affect the local connection. A negative value disables this functionality. | `600`
**blacklist** | Regular expression to deny IP addresses of incoming connections. | `"^$"`
**whitelist** | Regular expression to accept IP addresses of incoming connections. | `"127.0.0.1"`

___

## Obbus
### Description

This is responsible of offering a connection point with the pubsub of the system.

### Configuration example

    }
        ... other instances ...
        "obbus":{
            "module":	"obbus",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "address":	"0.0.0.0",
            "max_conns":	10,
            "port":	2324,
            "buffer_size":	37268,
            "timeout":	600,
            "blacklist":	"^$",
            "whitelist":	"^(10\\.|127\\.)"            
        },
        ... other instances ...
    }

### Specific module parameters

Parameter   | Description   | Default Value 
----------  | -----------   | ------------- 
**address** | IP address of the server. | `"127.0.0.1"`
**port** | Server port. | `2324`
**max_conns** | Maximum number of remote connections. | `10`
**buffer_size** | Buffer size of the obbus connections in bytes. | `37268`
**timeout** | Seconds of inactivity before closing the remote connection to prevent it from being permanently open. It does not affect the local connection. A negative value disables this functionality. | `600`
**blacklist** | Regular expression to deny IP addresses of incoming connections. | `"^$"`
**whitelist** | Regular expression to accept IP addresses of incoming connections. | `"127.0.0.1"`

___

## Slic-dahdi 
    }
        ... other instances ...
        "slic":	{
            "module":	"slic-dahdi",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "busytone":	"425/200,0/200",
            "channel":	1,
            "congestiontone":	"425/200,0/200,425/200,0/200,425/200,0/600",
            "dialtone":	"425",
            "dtmf_min_gain":	-40,
            "dtmf_min_gap":	50,
            "dtmf_rtwist":	-1,
            "dtmf_threshold":	-100,
            "dtmf_twist":	-1,
            "frmsize":	160,
            "lfr1":	-8,
            "lfr2":	-8,
            "ringtone":	"425/1500,0/3000"
        },
        ... other instances ...
    }
## Audiogen

    {
        ... other instances ...   
        "audiogen":	{
            "module":	"audiogen",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true
            "max_respawn_delay":	60,
	    }
        ... other instances ...
    }
## Gateway
    }
        ... other instances ...
        "gateway":{
            "module":	"gateway",
            "log_level":	"none",
            "autostart":	false,
            "respawn":	true,
            "max_respawn_delay":	60,
            "audiogen_dev":	"audiogen",
            "listen_addr":	"0.0.0.0",
            "listen_port":	2325,
            "modem_data_dev":	"modem",
            "modem_dev":	"modem",
            "modem_dev_gain":	0,
            "modem_dtmf_q_len":	30,
            "modem_dtmf_regen":	true,
            "modem_pcm_q_len":	2,
            "off_time":	15,
            "on_time":	100,
            "slic_dev":	"slic",
            "slic_dev_gain":	0,
            "slic_dtmf_q_len":	30,
            "slic_dtmf_regen":	true,
            "switch_polarity":	false,
            "trd1":	"=",
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
            "wait_digit":	4                  
        },
        ... other instances ...
    }
## Modem 
    {
        ... other instances ...   
    	"modem":	{
            "module":	"modem",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "dtmf_min_gain":	-40,
            "dtmf_min_gap":	85,
            "dtmf_rtwist":	-1,
            "dtmf_threshold":	-100,
            "dtmf_twist":	-1,
            "init_str":	"ATE0V1",
            "regen_dtmf_duration_ms":	100,
            "rx_frm_size":	800,
            "spk_vol":	7,
            "tty_cmd":	"/dev/ttyCMD",
            "tty_pcm":	"/dev/ttyPCM",
            "tx_frm_size":	320
	    },
    }
## Leds
    {
        ... other instances ...   
        "leds":	{
            "module":	"leds",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "led_ath":	"/sys/class/leds/ath9k-phy0",
            "led_lan":	"/sys/class/leds/gsr:green:lan",
            "led_usb":	"/sys/class/leds/gsr:red:usb",
            "led_wan":	"/sys/class/leds/gsr:green:wan",
            "led_wlan":	"/sys/class/leds/gsr:green:wlan",
            "switch_off":	"lan,wan,wlan,usb"
        },
        ... other instances ...   
    }
## Exec 
    {
        ... other instances ...
        "exec":	{
            "module":	"exec",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "max_in_size":	131072,
            "max_out_size":	131072,
            "min_out_size":	512,
            "timeout":	180
	    },
        ... other instances ...   
    }
## Report
    {
        ... other instances ...
        "report":	{
            "module":	"report",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "iface":	"eth0",
            "max_retry_time":	3600,
            "posturl":	"https://gsr-service-int.n4m.zone/hello",
            "timeout":	10
        },
        ... other instances ...   
    }
## Wdinet 
    {
        ... other instances ...
        "wdinet":	{
            "module":	"wdinet",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "awk_path":	"awk",
            "clean_untracked":	true,
            "head_path":	"head",
            "ifaces":	"eth0,eth1,wlan0,br-lan,ppp0",
            "ip_path":	"ip",
            "off_period":	30,
            "on_period":	300,
            "panic_cmd":	"reboot",
            "panic_timeout":	900,
            "ping_list":	"8.8.8.8,4.4.4.4",
            "ping_path":	"ping",
            "ping_tout":	4,
            "route_path":	"route"
        },
        ... other instances ...
    }
 
## 3g
    {
        ... other instances ...
        "3g":	{
            "module":	"3g",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "apn_file":	"/etc/apns.json",
            "force_apn":	"",
            "force_apn_pass":	"",
            "force_apn_user":	"",
            "metric":	30,
            "modem_init":	"ATE0V1",
            "panic_cmd":	"/usr/bin/panic_cmd.sh",
            "panic_timeout":	600,
            "pppd_params":	"mtu 1450",
            "pppd_path":	"pppd",
            "pppd_wait_iface":	25,
            "timeout":	5,
            "tty_hwc":	false,
            "tty_path":	"/dev/ttyPPP",
            "unit":	0
	    },
        ... other instances ...
    } 
## Sms 
    {
        ... other instances ...
        "sms":	{
            "module":	"sms",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "$ackbpass":	"telealarm.ack_bypass",
            "$ackdtmf":	"telealarm.ack_digits",
            "$acktout":	"telealarm.ack_timeout",
            "$apn":	"3g.force_apn",
            "$apnp":	"3g.force_apn_pass",
            "$apnu":	"3g.force_apn_user",
            "$ci":	"telealarm.test_call_interval",
            "$e0":	"telealarm.emerg0",
            "$e1":	"telealarm.emerg1",
            "$e2":	"telealarm.emerg2",
            "$e3":	"telealarm.emerg3",
            "$mda":	"telealarm.max_dial_attempts",
            "$saa":	"alert_acqcard.sms_to",
            "$sab":	"alert_battery.sms_to",
            "$sat":	"alert_telealarm.sms_to",
            "$sos0":	"telealarm.sos0",
            "$sos1":	"telealarm.sos1",
            "$swpl":	"gateway.switch_polarity",
            "$test0":	"telealarm.test0",
            "$test1":	"telealarm.test1",
            "$trd1":	"gateway.trd1",
            "$trd2":	"gateway.trd2",
            "$trd3":	"gateway.trd3",
            "$trd4":	"gateway.trd4",
            "$trd5":	"gateway.trd5",
            "$trd6":	"gateway.trd6",
            "$tro1":	"gateway.tro1",
            "$tro2":	"gateway.tro2",
            "$tro3":	"gateway.tro3",
            "$tro4":	"gateway.tro4",
            "$tro5":	"gateway.tro5",
            "$tro6":	"gateway.tro6",
            "modem_dev":	"modem",
            "password":	"12345",
            "respond":	true
	    },
        ... other instances ...
    }
## Battery
    {
        ... other instances ...
       	"battery":	{
            "module":	"battery",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "i2cbus":	"/dev/i2c-0",
            "reload_period":	60,
            "vars":	[{
                    "alert":	true,
                    "alert_low":	560,
                    "alert_safe":	590,
                    "name":	"v",
                    "thold":	2
                }, {
                    "alert":	false,
                    "name":	"i",
                    "thold":	2
                }, {
                    "alert":	true,
                    "alert_max":	50,
                    "name":	"t",
                    "thold":	2
                }, {
                    "alert":	false,
                    "name":	"soc",
                    "thold":	2
                }]
	    }, 
        ... other instances ...
    }
## Watchdog
    {
        ... other instances ...
        "watchdog":	{
            "module":	"watchdog",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true
            "max_respawn_delay":	60,
            "auto_close":	true,
            "auto_open":	true,
            "interval":	1,
            "path":	"/dev/watchdog",
	    },
        ... other instances ...
    }
## Vpn4m
    {
        ... other instances ...
        "n4m":	{
            "module":	"vpn4m",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "fail_cmd_error":	false,
            "host":	"eu1.z.n4m.zone",
            "if_name":	"n4m%d",
            "ip_cmd":	"/usr/sbin/ip",
            "ipt_cmd":	"/usr/sbin/iptables",
            "port":	7744,
            "user":	"user"
            "password":	"password",
	    },
        ... other instances ...
    }
## Evdev
    {
        ... other instances ...
        "evdev":	{
            "module":	"evdev",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true
            "max_respawn_delay":	60,
            "devpath":	"/dev/input/event0",
	    },
        ... other instances ...
    } 
## Alert
    {
        ... other instances ...
        "alert_battery":	{
            "module":	"alert",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "modem_dev":	"modem",
            "alert_path":	"battery",
            "max_ttl":	86400,
            "sms_to":	"",
            "timeout":	3600,
            "url_post":	"https://gsr-alerts-int.n4m.zone/alert/battery"
	    },
        ... other instances ...
    } 
## Button
    {
        ... other instances ...
        "button_config":	{
            "module":	"button",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true,
            "max_respawn_delay":	60,
            "actions":	[{
                    "payload":	"{\"cmd\":\"/usr/bin/btn_config.sh\", \"timeout\":120}",
                    "topic":	"exec.cmd",
                    "type":	"s"
                }],
            "code":	529,
            "keypress_max_secs":	6,
            "keypress_min_secs":	2,
            "type":	1
        },
        "button_factory":	{
            "module":	"button",
            "autostart":	true,
            "log_level":	"none",
            "respawn":	true,
            "max_respawn_delay":	60,
            "actions":	[{
                    "payload":	"{\"cmd\":\"/usr/bin/btn_reset_factory.sh\", \"timeout\":1200}",
                    "topic":	"exec.cmd",
                    "type":	"s"
                }],
            "code":	529,
            "keypress_max_secs":	60,
            "keypress_min_secs":	15,
            "type":	1
        },
        ... other instances ...
    }
## Gsrleds  
    {
        ... other instances ...
        "gsrleds":	{
            "module":	"gsrleds",
            "log_level":	"none",
            "autostart":	true,
            "respawn":	true
            "max_respawn_delay":	60,
            "battery_led":	"wan",
            "config_led":	"lan",
            "dial_led":	"lan",
            "modem_green_led":	"wlan",
            "modem_red_led":	"usb",
	    },
        ... other instances ...
    }