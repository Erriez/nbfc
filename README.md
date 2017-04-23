![NoteBook FanControl](https://github.com/hirschmann/nbfc/wiki/images/banner.png)

NBFC (NoteBook Fan Control) is a cross-platform Windows and Linux fan control service for notebooks.
It comes with a powerful configuration system, which allows to adjust it to many different notebook models.

## NBFC fork to support HP Pavilion 17-ab240nd 
This is a fork of the [NoteBook FanControl](https://github.com/hirschmann/nbfc) project to support the HP Pavilion 
17-ab240nd Notebook for Windows 10.

The HP Pavilion 17-ab240nd build-in CPU fan is automatically controlled by hardware which consists of 4 fan speeds:
* Step 1. On:              Noisy, but acceptable
* Step 2. Louder:          Annoying
* Step 3. Very loud:       Very annoying
* Step 4. Extremely loud:  Not acceptable at all

A "HP CoolSense Technology" Windows application v2.2 (August 14, 2014) is pre-installed, but the on/off feature has no
effect on the fan speed / cooling. The app is probable not compatible with the HP Pavilion 17-ab240nd (release date end 
of 2016) and can be safely uninstalled.

The BIOS contains a setting to turn the fan off by low temperatures. You can do this as follows:
1. Turn the laptop off
2. Hold down the ESC key and press the power-on button short.
3. Wait until a startup menu appears and select F10 to enter the BIOS.
4. Change the setting in the System Configuration menu | Fan Always On: Disabled
5. Press F10 to save the setting and reboot.

Now the CPU fan is controlled by hardware with the following temperature ranges:
1.	0           Fan speed 0: 0% (Fan off)
2.	50..37 `C	Fan speed 1: 25%
3.	55..38 `C	Fan speed 2: 50%
4.	60..42 `C	Fan speed 3: 75%
5.	68..45 `C	Fan speed 4: 100% (Fan max speed)

As you can see in this table, cooling down from maximum temperature results in large fan speed steps from loud to
silent. Only 5 fan speed steps are supported which seems to be a limitation of the hardware.

The hardware fan control is pretty noisy, so it's time to use a software fan control application, such as NBFC.

## NBFC Installation
Note: Windows 10 (with Creators Update) on the HP Pavilion 17-ab240nd requires NBFC v1.5.1 or higher.

## NBFC Usage
1. Start the NoteBook FanControl application from the Start menu.
2. Selected Config: "HP Pavilion 17-ab240nd"
3. The default setting is Read-only.
4. Check the temperature (Intel I7 CPU) for a valid temperature:
   * A normal temperature in idle is about 35..40 degree celsius. (No fan needed)
   * Hot around 65..70 degree -> Maximum fan speed is recommended.
   * Absolute maximum rating around 85 a 90 degree celsius, above will definitely damage the hardware.
5. The free HWiNFO64 application should display the same temperature in menu: 
   * CPU[#0]: Intel Core i7-7700HQ | CPU Package
6. Change to Enabled
7. Now you can drag the CPU Fan speed slider to test the fan speed in 5 steps.
   Make sure you don't overheat everything!
   Change to Auto for automatic fan control.
   
Note: "The Current fan speed" has no function and should be ignored.

## NBFC temperature - fan speed configuration
The fan control settings such as temperature and threshold should be modified with the Config Editor application.
1. Select the Config name: "HP Pavilion 17-ab240nd"
2. Go to the Fan configuration tab
3. Edit the CPU Fan line | tab: Temperature thresholds

The default temperature / fan speed values are safe within absolute maximum ratings to prevent damaging the notebook.
Feel free to experiment with the settings, but prevent overheating.

## Technical details
The "HP Pavilion 17-ab240nd" contains one 8-bit EC (Embedded Controller) register:
* Address 0x58 (88) read: i7 CPU package temperature in degree Celsius, automatically updated every 1..2 seconds.
* Address 0x58 (88) write: Fan speed, supported values:
    * 0x00..0x27: Fan speed 0: 0% (Fan off)
    * 0x28..0x37: Fan speed 1: 25%
    * 0x38..0x3C: Fan speed 2: 50%
    * 0x2D..0x43: Fan speed 3: 75%
    * 0x44..0x55: Fan speed 4: 100% (Fan max speed)

* The fan speed value is actually a temperature value which must be rewritten every 1000ms, otherwise the hardware will
  control the fan automatically.
* Do NOT write a value > 0x5A = 90 degree celsius, otherwise the notebook will immediately turned off to cool down. A 
  temperature failure message will be displayed on next boot. This incident is also reported in the BIOS log. (Fun for 
  HP support to see unexpected values)

A fan kick-start is generated in hardware. So when the fan is turned off and a new fan speed value is written, the 
hardware generates a high PWM to start the fan and decreases the fan speed slightly.

## Important EC notes: 
* Be careful with writing to EC registers, because writing incorrect values may damage the hardware.
* Reading from EC registers is no problem.
* Documentation of the EC peripheral is not available, so it requires reverse engineering at your own risk.
* A manufacture is free to develop their own EC implementation, so the registers are not compatible between different
  devices.

## EC measurement results
EC register dumped with the free Windows [RWEverything](http://rweverything.com/) on every increased fan speed step.
The free Prime95 application used to increase the CPU temperature by running CPU stress tests.
### Cold boot:
```
0x47=0x1C               # Seems to be 28 degree
0x58=0x29, 0x59=0x18    # Register 0x58 seems to be combined CPU package temp 41 degree (read) and fan speed (write)
0xB2=0x00, 0xB3=0x00    # Unknown, writing has no effect
```
### Fan speed 1:
```
0x47=0x24               # Seems to be 36 degree
0x58=0x2A, 0x59=0x1F    # Register 0x58 seems to be combined CPU package temp 42 degree (read) and fan speed (write)
0xB2=0xD3, 0xB3=0x07    # Unknown, writing has no effect
```
### Fan speed 2:
```
0x47=0x2C               # Seems to be 44 degree
0x58=0x37, 0x59=0x20    # Register 0x58 seems to be combined CPU package temp 55 degree (read) and fan speed (write)
0xB2=0xB9, 0xB3=0x09    # Unknown, writing has no effect
```
### Fan speed 3:
```
0x47=0x31               # Seems to be 49 degree
0x58=0x3C, 0x59=0x23    # Register 0x58 seems to be combined CPU package temp 60 degree (read) and fan speed (write)
0xB2=0xF3, 0xB3=0x0A    # Unknown, writing has no effect
```
### Fan speed 4:
```
0x47=0x3C               # Seems to be 60 degree
0x58=0x44, 0x59=0x2C    # Register 0x58 seems to be combined CPU package temp 68 degree (read) and fan speed (write)
0xB2=0x15, 0xB3=0x0C    # Unknown, writing has no effect
```

## Downloads
| Source | Link | Status |
|---|---|---|
| Download from github | [NBFC releases](https://github.com/hirschmann/nbfc/releases) | [![Github All Releases](https://img.shields.io/github/downloads/hirschmann/nbfc/total.svg)](https://github.com/hirschmann/nbfc/releases) |
|Install via chocolatey| [NBFC package info](https://chocolatey.org/packages/nbfc) | [![Chocolatey](https://img.shields.io/chocolatey/dt/nbfc.svg)](https://chocolatey.org/packages/nbfc) |

Currently there are no pre-built releases for Linux, but you can easily build NBFC yourself: [How to build NBFC](https://github.com/hirschmann/nbfc/wiki/How-to-build-NBFC)

## Getting started
- [First steps](https://github.com/hirschmann/nbfc/wiki/First-steps)
- [FAQ](https://github.com/hirschmann/nbfc/wiki/FAQ) (please read this before you submit a new issue)
- [Structure of a NBFC config file](https://github.com/hirschmann/nbfc/wiki/Structure-of-a-NBFC-config-file) (in case you want to edit or create a NBFC config)

## Credits
Many thanks to everyone who submitted pull requests, created config files, donated, or in any other way contributed to this project. :)
