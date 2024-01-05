# Table of Contents
* [Main](index.md)
* [Features](01_features.md)
* [Configuration](02_configuration.md)
* [Wiring](03_wiring.md)
* [NodeRED MQTT / HomeKit Example](04_nodered_example.md)
* [Home Assistant Example](05_homeassistant_example.md)
* [FAQ & Troubleshooting](09_faq.md)

## Home Assistant Example
If you are using Home Assistant, there are a couple of options for integration.

1. [ESP Home](http://github.com/ratgdo/esphome-ratgdo): The ESP Home version of ratgdo is directly adoptable by your Home Assistant instance.
2. [MQTT Integration](https://paulwieland.github.io/ratgdo/flash.html): If you are using Home Assistant and have an MQTT broker set up, HA will automatically discover ratgdo’s door, light, and obstruction sensors after configuration.

Ensure you review the [features matrix](https://paulwieland.github.io/ratgdo/01_features.html) before selecting an option. Not all options are available for every garage door opener.

### ESPHome
There is an ESPHome port of ratgdo available. For the time being, this port might not be feature-compatible with the MQTT version of ratgdo.

* [Web Tools installer](https://ratgdo.github.io/esphome-ratgdo/)
* [GitHub Repo](https://github.com/ratgdo/esphome-ratgdo)

### Dry Contacts
#### Triggers
##### Chamberlain / Liftmaster openers
When using either Security + 1.0 or Security + 2.0 door opener, ratgdo's dry contact triggers can be pulled to ground to trigger the door opener as follows:

- `open^1` - opens the door.
- `close^1` - closes the door.
- `light` - toggles the light on or off.

##### Other openers
When using a door opener that supports standard dry contact control (doorbell style), the trigger inputs are used to detect the door state. For these openers connect as follows:

- `trigger open terminal` - wire to door open limit switch
- `trigger close terminal` - wire to door closed limit switch

Wire it in such a way that the trigger input is connected with ground when the limit switch is closed. Some door openers (e.g., Genie) have screw terminals on them for each limit switch in addition to the control terminal for opening the door. If your opener doesn't expose its internal limit switches to user-accessible terminals, you can add simple reed switches to the door track to detect its state.

With these two limit switches connected, ratgdo can detect all four door states (closed, opening, open, closing).

#### Statuses
The following dry contact statuses are available:

- `door` - pulled to ground if the door is open, open circuit if closed.
- `obs` - pulled to ground if the door is obstructed, open circuit if clear.

### Notes
1. `^1` Repeated open commands (or repeated close commands) will be ignored. This gives discrete open/close control over the door, which is better than a toggle.

### MQTT
#### Home Assistant Auto Discovery
If you are using Home Assistant and have an MQTT broker set up, then HA will automatically discover ratgdo's door, light, and obstruction sensors after it boots up.

When ratgdo boots up after being configured, it broadcasts the necessary discovery messages that tell Home Assistant what it is and how to communicate with it.

See [Home Assistant](05_homeassistant_example.md) for more information.

#### Triggers
The following MQTT commands are supported:

- `command/door:open` - opens the door.
- `command/door:close` - closes the door.
- `command/light:on` - turns the light on.
- `command/light:off` - turns the light off.
- `command/lock:lock` - locks out the wireless remotes.
- `command/lock:unlock` - unlocks the use of wireless remotes.
- `command:query` - queries the door opener for current status.

##### Examples
If:

- Device Name = "MyGarageDoor"
- MQTT Prefix = "home/garage"

Then:

- `mqtt.topic = "home/garage/MyGarageDoor/command/door"; mqtt.payload = "open";` - opens the door
- `mqtt.topic = "home/garage/MyGarageDoor/command/door"; mqtt.payload = "close";` - closes the door

#### Statuses
The following statuses are broadcast over MQTT:

- `prefix/status/availability`
  - `online` - once ratgdo connects to the MQTT broker.
  - `offline` - the MQTT last will message which is broadcast by the broker when the ratgdo client loses its connection.
- `prefix/status/obstruction`
  - `obstructed` - when an object breaks the obstruction sensor beam.
  - `clear` - when an obstruction is cleared.
- `prefix/status/door`
  - `opening` - when the door is opening.
  - `open` - when the door is fully open.
  - `closing` - when the door is closing.
  - `closed` - when the door is fully closed.
- `prefix/status/light`
  - `on` - when the light is on.
  - `off` - when the light is off.
- `prefix/status/lock`
  - `locked` - when the door opener is locked.
  - `unlocked` - when unlocked.
