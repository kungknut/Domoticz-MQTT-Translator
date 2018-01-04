# Domoticz-MQTT-Translator

A translator used to translate MQTT-messages sent from or to Domotcz.

## What is it

This Node-Red flow is used to translate incoming/outgoing MQTT-messages to make any MQTT-enabled device compatible for use together with Domoticz.

## Features

- Full configuration via Domoticz user variables
- Communication with Domoticz via RESTful API.

## How it works

This flow reads the settings stored in Domoticz User Variables via its built-in JSON-API at startup or when triggered manually to allow full configuration via Domoticz web-interface.

## Installation and configuration

### Prerequisits

- You have a working install of Domoticz.
- You have a working install of Mosquitto or equivalent MQTT-broker.
- You have a working install of Node-Red.
- You have a MQTT-enabled IoT-device that you want to talk to. The
  device is sending (and receiving) information as a message via MQTT.

### Installation

Import this flow in Node-Red using the Import - Clipboard function. Copy the contents of `MQTT-Translator.json` and paste them in the textarea.

### Configuration

#### In Node-Red

First you will need change a couple of things to get this working together with your environment.

- Insert the correct IP-address and port of your Domoticz server in the
  following blocks (Default: `http://localhost:8080`);
  - "Read user variables"
  - "Convert to url"
- Insert the correct IP-address and port of your MQTT-broker in the
  following blocks (Default: `localhost:1883`);
  - "trigger"
  - "From 3rd party"
  - "From Domoticz"
  - "To 3rd party"

The functioning of this flow depends on the settings stored in Domoticz User Variables. The User Variable should be of type string and the settings should be formatted in Json as described in the example below. Order does not matter.

User variables are only read once at boot/start of Node-Red or when triggered manually either by sending a message to the MQTT-topic `nodered/update-uservar` or by clicking the trigger.

#### In Domoticz

Create a new User variable via Setup - More Options - User Variables of type string and populate it according to your needs.

```json
{
    "MQTT":{
        "devIdx" : 3,
        "inTopic" : "1327465/opsta",
        "outTopic" : "1327465/cmd",
        "slope" : 1,
        "intercept" : 0,
        "limits" : "0,0",
        "type" : "switch"
    }
}
```

##### devIdx (Required)

Domoticz ID of the device these settings correlate to, the same Idx must not be represented more than once.

##### inTopic (Optional, but this or `outTopic` must be present)

MQTT Topic used by the IoT device when sending data/commands to Domoticz. (Optional if the device do not send data/commands.)

##### outTopic (Optional, but this or `inTopic` must be present)

MQTT Topic used by the IoT device when receiving data/commands from Domoticz. (Optional if the device do not receive data/commands.)

##### slope, intercept (Optional)

These parameters are used to scale the value sent between Domoticz and the IoT-device. The value is handled according to the formulas below. Both parameters are optional. If omitted, **slope defaults to 1** and **intercept defaults to 0**. **From IoT to Domoticz:** `value * slope + intercept`. **From Domoticz to IoT:** `(value - intercept) / slope`.

##### limits (Optional)

This parameter is used to set certain limits of the value. What parameters to set depends on what type of device is configured.

When the value exceeds the first value stated, but not the second, the int paired with the first value will be used and so on. Separate pairs using `;`, there is no upper limit regarding the number of pairs allowed.

Syntax: `int,value;int,value;int,value`

##### type (Required)

This setting defines whether the IoT-device is a switch, dimmer etc.

| Value | Description |
| --- | --- |
| switch | IoT device handles `ON` or `OFF` messages, observe caps. |
| dimmer | IoT device handles a value of adjustable range, default 0-100, to set the dimming level. Refer to `slope` and/or `intercept` settings to adjust the range. |
| svalue | IoT device that is sending data to one of the following object types in Domoticz; `Temperature`, `Percent`, `Pressure`, `Voltage`, `Text` or `Distance`. |
| svalue1 | IoT device that is sending or receiving data to or from one of the following object types in Domoticz; `Thermostat setpoint`. |
| humidity | IoT device that is sending data to one of the following object types in Domoticz; `Humidity`, refer to the `limits` parameter to set Wet, Dry etc. Example below |

Setting up limits for humidity device;

Use the "limits" parameter to define the humidity status limits (Optional, defaults to Normal).

- 0 = Normal
- 1 = Comfortable
- 2 = Dry
- 3 = Wet

Example:

`"limits":"2,0;1,25;3,75"` gives 0-24% = Dry, 25-74% = Comfortable, 75-100% Wet

Example2, if you put the digits in the wrong order, eg below, the latest will win:

`"limits":"1,25;2,0;3,75"` gives 0-74% = Dry, 75-100% Wet

## Versioning

This project follows [Semantic Versioning 2.0](https://semver.org/) and uses version numbers like `MAJOR`.`MINOR`.`PATCH`.

## Contributing

Anyone can contribute to improve or fix, to do so you can either report an issue (a bug, an idea...) or fork the repository, perform modifications to your fork then request a merge.