# midas-pro-series-osc-commands
This repository contains the commands and information necessary to control a pro-series (such as the pro2c) mixer from Midas, over OSC. 


The commands were partially found by reverse-engineering and disassemblying the firmware files of a Midas pro-series mixer (sorry, Midas). The resulting list of paremeter nodes were then brute-forced on a poor pro2c for about 24 hours until all working combinations were found. After this, many of them have been manually tested in order to figure out their real purpose, and meaningful descriptions have been written.

This repository consists of two main files:
- `pro-series-endpoints.json`: Contains most OSC commands
- `pro-series-internal-fx-parameters-endpoints.json`: Contains all internal FX read/write commands. Split up as these were 700 commands alone.

## Command syntax
The pro-series OSC commands usually consist of an OSC command consisting of three parts:
1. The command type, one of:
    - `enPPCFaderMessage`: A float between 0 and 1 (but greater than 0 and less than 1, so min value is something like 0.0000001 and max value is 99.9999999). Used for most fader-based controls.
    - `enPPCRotaryMessage`: Similar to enPPCFaderMessage. More widely used, and supports values of 0 and 1. Please note that as this message type only supports a float value between 0 and 1, you will have to create functions that parse an input value (e.g. 20hz if the parameter you want to control is a frequency) into the correct float value (e.g. 0.22). This specification will eventually have this information as well, which is for now included in the description of some commands.
    - `enPPCStringMessage`: Self-explanatory. Used for anything text-related, such as changing the labels for an input.
    - `enPPCMeterMessage`: A read-only command for reading out meters. Returns a float value between 0 and 1, and as such, is subject to the same hurdles as enPPCRotaryMessage.
    - `enPPCSwitchMessage`: A boolean message, represented as an integer of 0 and 1. Nearly all of these are toggles, where sending an integer of 1 will toggle the switch.
    - `enPPCOtherMessage`: I'm not yet sure what this message type does yet.
2. The group of message. For example `enPPCVirtualMicInputs`.
3. The paremeter node you want to change. For example `enFaderLevel`.

Last, but not least, the command may also end with an integer, if the command can control multiple instances of a thing (e.g. multiple microphone inputs). For `enVirtualMicInputs` this is the index of the microphone input you want to change. Please note that this is 0-indexed, so input 1 is index 0 (`/enPPCFaderMessage/enVirtualMicInputs/enFaderLevel/0`).

All commands support both getting and setting a value. To get a value, you simply send the command without any arguments. To set a value, you must send the command with the arguments you wish to set.

## JSON specification syntax
The pro series features a massive amount of parameters to control, and as such, a massive amount of commands have been found on OSC. It's not clear what they all do, and some are mere duplicates of each other but with a slight twist. Some even have typos in them. I have therefore made a simple JSON file with the currently discovered commands in order to keep it organized.

The JSON file contains an object that contains the control groups. This object then contains all the parameter nodes that you can get and/or set.

Each node contains some documentation in the following format:
- `multiPath`: Whether or not the node contains multiple instances. All nodes in `enVirtualMicInputs` do, but some nodes in other groups may only control one thing.
- `type`: The command type (e.g. `enPPCFaderMessage`)
- `argumentType`: The OSC argument type. If this is set to `null`, it means that the node is not set-able. All `enPPCMeterMessage` command types have this set to `null`.
- `description`: A human-readable description of what the command does. In some instances this may only show the parameter node name, in which case I haven't yet figured out what that command does.
- `isAbsolute`: This is currently set to `true` for the ONE `enPPCSwitchMessage` command that isn't toggle. If it's `undefined` just assume the command is toggle. This is also reflected in the parameter node description.

## How to use
You will need to send the commands using OSC. I recommend using the `osc` package for node.js.
You will also need to set an IP address on your pro mixer and enable ethernet control in the settings.

## What's next?
This was created in about a week, so expect updates to the specification to come swiftly.
