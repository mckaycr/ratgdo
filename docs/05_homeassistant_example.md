#### Table of Contents
* [Main](index.md)
* [Features](01_features.md)
* [Configuration](02_configuration.md)
* [Wiring](03_wiring.md)
* [NodeRED MQTT / HomeKit Example](04_nodered_example.md)
* [Home Assistant Example](05_homeassistant_example.md)
* [FAQ & Troubleshooting](09_faq.md)

# Home Assistant Example
If you're using Home Assistant, there are a couple of options for integration:

1. [ESP Home](http://github.com/ratgdo/esphome-ratgdo): The ESP Home version of ratgdo is directly adoptable by your Home Assistant instance.
2. [MQTT Integration](https://paulwieland.github.io/ratgdo/flash.html): For Home Assistant users with an MQTT broker set up, ratgdo’s door, light, and obstruction sensors will be automatically discovered by HA after configuration.

Before making a choice, review the [features matrix](https://paulwieland.github.io/ratgdo/01_features.html). Not all options are available for every garage door opener, and using the wrong one may result in random operations with your garage door.

## ESPHome
_For the time being, avoid using this firmware if your Chamberlain Liftmaster has the purple, red, or orange learn buttons._

There is an ESPHome port of ratgdo available. However, it may not be fully compatible with the MQTT version of ratgdo at the moment.

To get started, use the [Web Tools installer](https://ratgdo.github.io/esphome-ratgdo/) to flash your ratgdo. Additional documentation can be found on the [GitHub Repo](https://github.com/ratgdo/esphome-ratgdo).

### Dry Contacts
#### Triggers
##### Chamberlain / Liftmaster openers
When using Security + 1.0 or Security + 2.0 door opener, ratgdo's dry contact triggers can be pulled to ground to trigger the door opener as follows:
- `open`<sup>1</sup> - opens the door.
- `close`<sup>1</sup> - closes the door.
- `light` - toggles the light on or off.

##### Other openers
For door openers supporting standard dry contact control (doorbell style), connect as follows:
- `trigger open terminal` - wire to door open limit switch
- `trigger close terminal` - wire to door closed limit switch

Wire it so that the trigger input is connected with ground when the limit switch is closed. Some openers (e.g., Genie) have screw terminals for each limit switch, in addition to the control terminal for opening the door. If your opener doesn't expose its internal limit switches to user-accessible terminals, add reed switches to the door track to detect its state.

With these two limit switches connected, ratgdo can detect all four door states (closed, opening, open, closing).

#### Statuses
The following dry contact statuses are available:
- `door` - pulled to ground if the door is open, open circuit if closed.
- `obs` - pulled to ground if the door is obstructed, open circuit if clear.

### Notes
<sup>1</sup> Repeated open commands (or repeated close commands) will be ignored. This provides discrete open/close control over the door, which is better than a toggle.

## MQTT
### Home Assistant Auto Discovery
If you use Home Assistant with an MQTT broker, use this feature for setting up your ratgdo device.

To get started, use the [ratgdo web installer for MQTT integrations](https://paulwieland.github.io/ratgdo/flash.html). This installer will load the MQTT firmware, help you connect to your WiFi, and provide you a "visit" link to the ratgdo configuration page for adding your MQTT configuration.

When ratgdo boots up after being configured, it broadcasts the necessary discovery messages telling Home Assistant what it is and how to communicate with it. Find your device listed under the MQTT integration in Home Assistant, where you'll have a "visit" link in case you need to adjust your MQTT configuration.

On the ratgdo configuration page, there's a place to change the Home Assistant auto-discovery prefix. The default is `homeassistant`, which is what HA uses out of the box. Change this setting to match your Home Assistant’s prefix if you've modified the standard value in HA’s MQTT integration settings.

### Configuration
This section only applies to the standard version, which uses WiFi & MQTT. The No WiFi version does not have any configuration options.

After the firmware is flashed, the ratgdo will reboot, ask you for your WiFi information, and attempt to connect to your WiFi. Once this is successful, click "Visit Device" from the ESP Web Tools page, and you will be presented with ratgdo's configuration page. 

_**Security Note:**
After the initial configuration is made, the device's config page will be loosely protected by an HTTP basic authentication challenge. The username is the device name (case sensitive), and the password is the OTA Password that you specified within the configuration page. For stronger protection, check the _Disable OTA_ checkbox, and the firmware will completely disable the HTTP web & ArduinoOTA services. With these services disabled, the only way to update the configuration is to reflash the firmware with a USB cable._

**Configuration Fields:**

- **Device Name** - Must be unique. Used as the device's hostname on the network, the MQTT device name, part of the path on the MQTT Topic (see MQTT prefix below), and the username for basic authentication to the ESP's built-in web server. DO NOT use spaces or illegal characters in the name.
- **OTA Password** - Used for Over The Air firmware flashing and as the HTTP basic authentication password for the web server configuration page. Use a very strong password if you leave the OTA/web service enabled since anyone with access to ratgdo over the network could also gain control over your garage door.
- **IP Address** - Will be filled in automatically with the DHCP address but can also be set manually.
- **SSID** - Your WiFi network name (case sensitive).
- **WiFi Password** - The password for the WiFi.
- **Enable MQTT** - Leave checked. If you do not wish to use MQTT, there is no reason to use the WiFi version of the firmware.
- **MQTT Server IP** - Required. The address of your MQTT broker.
- **MQTT Server Port** - Required. The port used for MQTT communication. 1883 is the default port.
- **MQTT Server Username & Password** - Optional. The Username & Password for authentication to the MQTT broker.
- **MQTT Topic Prefix** - Required. This is the prefix used for creating the MQTT topic that ratgdo will publish & subscribe to. `TOPIC_PREFIX/DEVICE_NAME/[command|status]`
    - Example: With Device Name of `MainDoor` and a topic prefix of `/home/garage/`, ratgdo will subscribe to MQTT topic `/home/garage/MainDoor/command`, and it will publish to `/home/garage/MainDoor/status`. _Note:_ If you are using Home Assistant, do not put a space or any "illegal" characters in your device name or prefix; otherwise, HA will not be able to add the device.
- **Home Assistant Discovery Prefix** - `homeassistant` is the default prefix that Home Assistant uses. If you changed your HA auto discovery configuration, then update this setting.
- **Disable OTA & Web Server Config Access** - Will disable the ArduinoOTA & Web service for additional security. If you use this option, you will have to reflash the firmware with a USB cable to change any config settings. 
- **Control Protocol** - Required. 
	- Chamberlain / LiftMaster with YELLOW learn button (Except some wall-mounted jackshaft openers): Choose Security + 2.0
	- Chamberlain / LiftMaster with PURPLE or RED learn button (And some wall-mounted jackshaft openers): Choose Security + 1.0
	- Other openers - choose Dry Contact

### Triggers
The following MQTT commands are supported:
- `command/door:open` - opens the door.
- `command/door:close` - closes the door.
- `command/light:on` - turns the light on.
- `command/light:off` - turns the light off.
- `command/lock:lock` - locks out the wireless remotes.
- `command/lock:unlock` - unlocks the use of wireless remotes.
- `command:query` - queries the door opener for the current status.

#### Examples
If:
- Device Name = "MyGarageDoor"
- Ratgdo Configuration MQTT Prefix = "garage"
- HA Discovery Prefix = "homeassistant"
Then:
- `mqtt.topic = "homeassistant/garage/MyGarageDoor/command/door"; mqtt.payload = "open";` - opens the door
- `mqtt.topic = "homeassistant/garage/MyGarageDoor/command/door"; mqtt.payload = "close";` - closes the door

### Statuses
The following statuses are broadcast over MQTT:
- `prefix/status/availability`
  - `online` - once ratgdo connects to the MQTT broker.
  - `offline` - the MQTT last will message, broadcast by the broker when the ratgdo client loses its connection.
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
