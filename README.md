![Logo](admin/iot.png)
# ioBroker MQTT cloud adapter
=================

[![NPM version](http://img.shields.io/npm/v/iobroker.iot.svg)](https://www.npmjs.com/package/iobroker.iot)
[![Downloads](https://img.shields.io/npm/dm/iobroker.iot.svg)](https://www.npmjs.com/package/iobroker.iot)

[![NPM](https://nodei.co/npm/iobroker.iot.png?downloads=true)](https://nodei.co/npm/iobroker.iot/)

This adapter allows connection from internet through ioBroker cloud to local installation of ioBroker.

## Settings
To use cloud adapter you should first to register on the iobroker cloud [https://iobroker.pro](https://iobroker.pro).

![Intro](img/intro.png)

### Language
If you select "default" language the smart names of devices and of enumerations will not be translated. If some language specified all known names will be translated into this language.
It is done to switch fast between many languages for demonstration purposes.

### Place function in names first
Change the order of function and roles in self generated names:

- if false: "Room function", e.g. "Living room dimmer"
- if true: "Function room", e.g. "Dimmer living room"

### Concatenate words with
You can define the word which will be placed between function and room. E.g. "in" and from "Dimmer living room" will be "Dimmer in living room".

But is not suggested to do so, because recognition engine must analyse one more word and it can lead to misunderstandings.

### OFF level for switches
Some groups consist of mixed devices: dimmers and switches. It is allowed to control them with "ON" and "OFF" commands and with percents.
If command is "Set to 30%" and the *OFF level" is "30%" so the switches will be turned on. By command "Set to 25%" all switches will be turned OFF.

Additionally if the command is "OFF", so the adapter will remember the current dimmer level if the actual value is over or equal to the "30%".
Later when the new "ON" command will come the adapter will switch the dimmer not to 100% but to the level in memory.

Example:

- Assume, that *OFF level* is 30%.
- Virtual device "Light" has two physical devices: *switch* and *dimmer*.
- Command: "set the light to 40%". The adapter will remember this value for *dimmer*, will set it for "dimmer" and will turn the *switch* ON.
- Command: "turn the light off". The adapter will set the *dimmer* to 0% and will turn off the *switch*.
- Command: "turn on the light". *dimmer* => 40%, *switch* => ON.
- Command: "set the light to 20%". *dimmer* => 20%, *switch* => OFF. The value for dimmer will not be remembered, because it is bellow *OFF level*.
- Command: "turn on the light". *dimmer* => 40%, *switch* => ON.

### by ON
You can select the behaviour of ON command will come for the number state. The specific value can be selected or last non zero value will be used.

### Write response to
For every command the text response will be generated. You can define here the Object ID , where this text must be written to. E.g. *sayit.0.tts.text*.

### Colors
Just now only english alexa supports the color control.
The channel must have 4 states with following roles:

- level.color.saturation (required for detection of the channel),
- level.color.hue,
- level.dimmer,
- switch (optional)

```
Alexa, set the "device name" to "color"
Alexa, turn the light fuschia
Alexa, set the bedroom light to red
Alexa, change the kitchen to the color chocolate
```

### Lock
To have the possibility to lock the locks, the state must have the role "switch.lock" and have native.LOCK_VALUE to determine the lock state.

```
Alexa, is "lock name" locked/unlocked
Alexa, lock the "lock name"
```

## How names will be generated
The adapter tries to generate virtual devices for smart home control (e.g. Amazon Alexa or Google Home).

The are two important enumerations for that: rooms and functions.

Rooms are like: living room, bath room, sleeping room.
Functions are like: light, blind, heating.

Following conditions must be met to get the state in the automatically generated list:

- the state must be in some "function" enumeration.
- the state must have role ("state", "switch" or "level.*", e.g. level.dimmer) if not directly included into "functions".
It can be that the channel is in the "functions", but state itself not.
- the state must be writable: common.write = true
- the state dimmer must have common.type as 'number'
- the state heating must have common.unit as '°C', '°F' or '°K' and common.type as 'number'

If the state is only in "functions" and not in any "room", the name of state will be used.

The state names will be generated from function and room. E.g. all *lights* in the *living room* will be collected in the virtual device *living room light*.
The user cannot change this name, because it is generated automatically.
But if the enumeration name changes, this name will be changed too. (e.g. function "light" changed to "lights", so the *living room light* will be changed to *living room lights*)

All the rules will be ignored if the state has common.smartName. In this case just the smart name will be used.

if *common.smartName* is **false**, the state or enumeration will not be included into the list generation.

The configuration dialog lets comfortable remove and add the single states to virtual groups or as single device.
![Configuration](img/configuration.png)

If the group has only one state it can be renamed, as for this the state's smartName will be used.
If the group has more than one state, the group must be renamed via the enumeration's names.

To create own groups the user can install "scenes" adapter or create "script" in Javascript adapter.

### Replaces
You can specify strings, that could be automatically replaced in the devices names. E.g if you set replaces to:
```.STATE,.LEVEL```, so all ".STATE" and ".LEVEL" will be deleted from names. Be careful with spaces.
If you will set ```.STATE, .LEVEL```, so ".STATE" and " .LEVEL" will be replaced and not ".LEVEL".

## Helper states
- **smart.lastObjectID**: This state will be set if only one device was controlled by home skill (alexa, google home).
- **smart.lastFunction**: Function name (if exists) for which last command was executed.
- **smart.lastRoom**:     Room name (if exists) for which last command was executed.
- **smart.lastCommand**:  Last executed command. Command can be: true(ON), false(OFF), number(%), -X(decrease at x), +X(increase at X)
- **smart.lastResponse**: Textual response on command. It can be sent to some text2speech (sayit) engine.

## IFTTT
[instructions](doc/ifttt.md)

## Services
There is a possibility to send messages to cloud adapter.
If you call ```[POST]https://iobroker.net/service/custom_<NAME>/<user-app-key>``` und value as payload.

```
curl --data "myString" https://iobroker.net/service/custom_test/<user-app-key>
```

If you set in the settings the field "White list for services" the name *custom_test*, and call with "custom_test" as the service name, the state **cloud.0.services.custom_test** will be set to *myString*.

You may write "*" in white list and all services will be allowed.

From version 2.0.5 you can use GET request in form ```[GET]https://iobroker.net/service/custom_<NAME>/<user-app-key>/<data>``` to place the **\<data\>** into **cloud.0.services.custom_\<NAME\>**.

Here you can find instructions how to use it with [tasker](doc/tasker.md).

IFTTT service is allowed only if IFTTT key is set.

Reserved names are "ifttt", "text2command", "simpleApi", "swagger". These must be used without the ```"custom_"``` prefix.

### text2command
You may write "text2command" in white list, you can send POST request to ```https://iobroker.net/service/text2command/<user-app-key>``` to write data into *text2command.X.text* variable.

"X" can be defined in settings by the "Use text2command instance" option.

## Changelog
### 0.1.0 (2018-09-17)
* (bluefox) Initial commit

## License
The MIT License (MIT)

Copyright (c) 2018 bluefox <dogafox@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

