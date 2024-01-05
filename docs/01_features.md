#### Table of Contents
- [Main](index.md)
- [Features](01_features.md)
- [Wiring](03_wiring.md)
- [NodeRED MQTT / HomeKit Example](04_nodered_example.md)
- [Home Assistant Example](05_homeassistant_example.md)
- [FAQ & Troubleshooting](09_faq.md)
# Features
If you have a security + 2.0 door opener, or a security + 1.0 opener with a compatible wall control panel, ratgdo detects the garage door's position (opening, open, closing, closed) from the encrypted signal wire. No soldering or additional sensors are required to get the door status. Three simple wires (Ground, Control, and Obstruction) are connected to the terminals of the garage door opener.

If you have a dry contact control door opener (e.g., Genie, old Chamberlain, etc.), then ratgdo can control your door and detect the door's position using two simple reed switches (not included).

Additionally ratgdo interfaces with Home Assistant, NodeRed, and HomeKit

## Feature Matrix
The features supported depend on the type of garage door opener you have and the firmware you are using. This table is meant to help clarify what features work with which opener.

Firmware types:

- **[M](http://github.com/ratgdo/mqtt-ratgdo)**QTT - for MQTT-based home automation integration (NodeRED, Home Assistant, etc)
- **[E](http://github.com/ratgdo/esphome-ratgdo)**SP Home - for ESP Home / Home Assistant
- **[H](http://github.com/ratgdo/homekit-ratgdo)**omeKit - (in development) if you just want iOS integration without the need for a home automation platform

|                | Manufactured by Chamberlain / Liftmaster |                | Other         |
|----------------|----------------------------------------|----------------|---------------|
|                | Security + 2.0                         | Security + 1.0 | Dry Contact   |
|                | Yellow Learn Button<sup>5</sup>        | Purple, Red, Orange Learn Buttons |           |
| **Firmware**   | <center>[M](http://github.com/ratgdo/mqtt-ratgdo) [E](http://github.com/ratgdo/esphome-ratgdo) [H](http://github.com/ratgdo/homekit-ratgdo)</center> | <center>[M](http://github.com/ratgdo/mqtt-ratgdo) [E](http://github.com/ratgdo/esphome-ratgdo) [H](http://github.com/ratgdo/homekit-ratgdo)</center> | <center>[M](http://github.com/ratgdo/mqtt-ratgdo) [E](http://github.com/ratgdo/esphome-ratgdo) [H](http://github.com/ratgdo/homekit-ratgdo)</center> |
| Door Control | X X X | X + + | X + + |
| Door Status  | X X X| X/O<sup>4</sup> +<sup>4</sup> +<sup>4</sup>| o<sup>1</sup> +<sup>1</sup> +<sup>1</sup> |
| Light Control | X X X | X + + |   |
| Light Status  | X X X | X + + |   |
| Obstruction Status | X X X | X + + | o<sup>2</sup> +<sup>2</sup> +<sup>2</sup> |
| Wireless Remote Lockout | X X X | X + + |   |
| Motion Detection | o<sup>3</sup> o<sup>3</sup> +<sup>3</sup>  |   |

<sup>1</sup> Openers with dry contact control require that limit switches be connected to ratgdo to detect the door state. See [Dry Contact Wiring](03_wiring.md).

<sup>2</sup> Obstruction sensors must have a peak voltage between 4.5 and 7 volts.

<sup>3</sup> Motion detection requires a wall control panel with a built-in motion detector such as the 889LM.

<sup>4</sup> Security + 1.0 openers can report door status over the data line, but not all wall panels are compatible. ratgdo listens for a wall panel to communicate with the door, and if it detects one (such as an [889LM](https://www.google.com/search?q=889lm+chamberlain)), it listens and reports the door status. If ratgdo doesn't hear wall panel communication, it switches to emulation mode, where it streams the query commands necessary to get the door opener status. Emulation mode will cause analog wall panels (e.g., [78LM](https://www.google.com/search?q=78LM+chamberlain)) to not be able to control the lights or lockout the wireless remotes because their analog commands will be ignored by the door opener.

<sup>5</sup> All yellow learn button openers EXCEPT the jackshaft wall-mounted 8500/RJ020 & 8500C/RJ020C are Security + 2.0. The 8500/RJ020 & 8500C/RJ020C use Security + 1.0.
