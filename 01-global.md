# Global pubsub topics

> When documenting the topics, sometimes the description states that the topic "returns" some value or error. This means that providing a response topic, a message with that value or error (with the corresponding *Error* flag) will be published to that response topic.
>
> If no return value or error is documented, the topic doesn't respond.

## Instance topics

All the instances have the following topics available: 

Topic | Type | Description
:--- | :--- | :---
`instance_name`.start | (ignored) | Publish to start the `instance_name`. An `"ok"` string is returned.
`instance_name`.stop | (ignored) | Publish to stop the `instance_name`. An `"ok"` string is returned.
`instance_name`.restart | (ignored) | Publish to restart (stop and start) the `instance_name`. An `"ok"` string is returned.
`instance_name`.reload | (ignored) | Publish to force the `instance_name` to reload the configuration. This is useful if the instance is in the `started` state, if configuration parameters were modified (through `sys.cmd.cfg.set` or `sys.cmd.cfg.reload`) this will make the instance to use the new values and restart itself if required. An `"ok"` string is returned.
`instance_name`.qstat | (ignored) | Publish to force the `instance_name` to publish it's state at the topic `instance_name`.evt.mod.stat` 
`instance_name`.evt.mod.stat | Integer | Subscribe to receive the state code of the `instance_name` when it changes.

See details on **Instance lifecycle** section for each state code and the effect of publishing to `start`, `stop` and `restart`.

___

## System topics

The obelisk system itself offers some topics to control it.

### Log level control

Topic | Type | Description 
:--- | :--- | :---
sys.cmd.loglevel | String | Publish to set the main log level of the obelisk system, one of: `"debug"`, `"info"`, `"notice"`, `"warning"`, `"error"`, `"crit"`, `"alert"`, `"emerg"`.

### Configuration control

Topic | Type | Description 
:--- | :--- | :---
sys.cmd.cfg.get | (ignored) | Publish to this topic with a response topic, all the configuration as a JSON string will be returned.
sys.cmd.cfg.get.`instance_name` | (ignored) | Publish to this topic with a response topic, all the configuration of the `instance_name` as a JSON string will be returned (or a string error if the instance is not found).
sys.cmd.cfg.get.`instance_name`.`parameter` | (ignored) | Publish to this topic with a response topic, the value of `parameter` of the `instance_name` as a JSON string will be returned (or a string error if the instance or the parameter is not found).
sys.cmd.cfg.set | String | Publish a string containing all the configuration in JSON to set it, the old configuration is discarded. An `"ok"` string is returned (or a descriptive string error).
sys.cmd.cfg.set.`instance_name` | String | Publish a string containing all the configuration of the `instance_name` in JSON to set it, the old configuration is discarded. An `"ok"` string is returned (or a descriptive string error). 
sys.cmd.cfg.set.`instance_name`.`parameter` | String | Publish a string containing the JSON value to set the `parameter` of the `instance_name`. An `"ok"` string is returned (or a descriptive string error). 
sys.cmd.cfg.unset | (ignored) | Publish to clear all the configuration. An empty map will replace the old configuration. An `"ok"` string is returned.
sys.cmd.cfg.unset.`instance_name` | (ignored) | Publish to delete the `instance_name` from the configuration if it exists. An `"ok"` string is returned.
sys.cmd.cfg.unset.`instance_name`.`parameter` | (ignored) | Publish to delete the `parameter` from the `instance_name` configuration if they exist. An `"ok"` string is returned.
sys.cmd.cfg.save | (ignored) | Publish to save the current state of configuration to the JSON file to make it persistent. An `"ok"` string is returned (or a descriptive string error).
sys.cmd.cfg.reload | (ignored) | Publish to load the configuration from the JSON file again. This discards the previous not saved changes. An `"ok"` string is returned (or a descriptive string error).

### Lifecycle control

Topic | Type | Description 
:--- | :--- | :---
sys.cmd.restart | (ignored) | Publish to restart obelisk. It will stop all the instances and clean their resources, reload the configuration and start from zero.
sys.cmd.reload | (ignored) | Publish to reload all the instances with the new configuration. The instances that no longer exist will be stopped and destroyed; the new instances will be created (and autostarted if configured to do so); the instances that already existed and continue existing will be "reloaded" to force them to use their new configuration values (see `instance_name`.reload).