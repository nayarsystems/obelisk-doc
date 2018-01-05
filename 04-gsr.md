# Obelisk running on GSR

The GSR device (GSM Smart Router) uses obelisk as its core. The following section describes de details on how is it run and the environment provided.

## Operating system

The GSR runs an image based on [LEDE](https://lede-project.org/), a GNU/Linux operating system based on OpenWrt.

## Running obelisk

The command `gsr` is a symlink to `obelisk`, both located on `/usr/bin`.

The `obelisk-wrapper` script (located on `/usr/bin`) is responsible for launching and restarting `obelisk` if it dies. It is automatically started when the device boots with the `/etc/init.d/obelisk` service. This service provides `/etc/init.d/obelisk stop`, `/etc/init.d/obelisk start` and `/etc/init.d/obelisk restart` commands for managing obelisk.

The `obelisk-wrapper` script starts obelisk with the **main config file** `/etc/obelisk/obelisk.json` (where the changes to the config are saved) on top of a **failsafe config file** `/etc/obelisk/failsafe.json` that is never modified. This ensures that some modules are always enabled and a minimum functionality is preserved in case of disaster (for example when saving an empty config or when accidentally deleting the config file).

If `obelisk` crashes, `obelisk-wrapper` launches it again. If 5 consecutive crashes occur and each running time has been inferior to 5 minutes it probably means that something is wrong in the configuration or the binary and `obelisk-wrapper` enters the failsafe mode.

The **failsafe mode** tests if the binary can be run, if it can't obelisk is run from the factory read-only binary (`/rom/usr/bin/obelisk`) and the main config file is restored to factory defaults (`/rom/etc/obelisk/obelisk.json`).

In failsafe mode, the failsafe config file is loaded and its defined instances started. Then the main config file is loaded but its instances are not started. Modifications done to the config while obelisk is running are done taking into account the main config file and then, when the config is saved, this is done to the main config file. When this happens (or after 10 minutes) the device reboots. The failsafe mode allows to externally connect to fix things or to reset the configuration to some known good values.

> Take special care of stopping `obelisk` when connected through a VPN connection that it handles. The connection will stop and `obelisk` won't be started again.
> Connecting through the LAN interface doesn't present this risk.

## Reading the logs

To read the syslog where obelisk logs all it's data, the `logread` command must be used.

`logread -f -l 200` shows the last 200 lines and follows the output.

## Custom scripts

There is some free space for users to add their custom scripts wherever they want and interface with obelisk through a local connection to obbus.

You can download the toolchain needed to compile software here: http://gsr-repo.n4m.zone/repo/external/toolchains/v3/Nayar-Toolchain-mips-24kc_gcc-6.3.0_musl-1.1.16.Linux-x86_64.tar.bz2

We also provide a Lua 5.1 environment, but you will need to install socket and msgpack packages manually to be able to connect to obbus, since these are not installed by default to save some space.

> Be aware that there is only ~3MiB of storage available to host your custom scripts and any data you may want to preserve

You will also need to setup a startup script on `/etc/init.d/` to run your scripts on boot alongside obelisk
