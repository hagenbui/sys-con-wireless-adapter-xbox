# sys-con + Xbox Wireless Adapter

**Use Xbox One and Series controllers wirelessly on a modded Nintendo Switch, through Microsoft's Xbox Wireless Adapter for Windows.**

This is a fork of [sys-con](https://github.com/o0Zz/sys-con) by o0Zz. It adds one thing: a driver for the Xbox Wireless Adapter, so controllers without Bluetooth (such as the original Xbox One controller) work on the Switch without a cable. Everything else sys-con does still works the same.

[![Download](https://img.shields.io/github/v/release/hagenbui/sys-con-wireless-adapter-xbox?label=download&style=for-the-badge)](https://github.com/hagenbui/sys-con-wireless-adapter-xbox/releases/latest)

## Tested on

| | |
|---|---|
| Console | Nintendo Switch with Atmosphère, system firmware 19.0.1 |
| Adapter | Xbox Wireless Adapter for Windows, slim model (`045e-02fe`) |
| Controller | Xbox One controller without Bluetooth |
| Connection | Adapter plugged into the dock |
| Working | Buttons and sticks, rumble, Home button, reconnects after the console sleeps |

**Not tested yet:** the original large adapter (`045e-02e6`), Xbox Series X\|S and Elite controllers, more than one controller at a time, handheld mode through a USB-C adapter. They are expected to work; if you try one, please [open an issue](https://github.com/hagenbui/sys-con-wireless-adapter-xbox/issues) either way.

## Install

You need a Switch running [Atmosphère](https://github.com/Atmosphere-NX/Atmosphere) custom firmware.

1. **Download** the zip from the [latest release](https://github.com/hagenbui/sys-con-wireless-adapter-xbox/releases/latest).
2. **Copy** everything inside the zip to the root of your SD card, replacing existing files. This replaces `/config/sys-con/config.ini`; back it up first if you changed it.
3. **Add the adapter firmware** to `/config/sys-con/` on the SD card, named `xbox_adapter_firmware.bin` (see below).
4. **Reboot** the Switch.

### Getting the adapter firmware

The adapter has no permanent firmware: the host loads Microsoft's firmware into it every time it is plugged in. That file belongs to Microsoft, so it is not included here. Get it from Microsoft's own driver package:

1. Download the driver package from Windows Update:
   [`1cd6a87c-623f-4407-a52d-c31be49e925c_e19f60808bdcbfbd3c3df6be3e71ffc52e43261e.cab`](http://download.windowsupdate.com/c/msdownload/update/driver/drvs/2017/07/1cd6a87c-623f-4407-a52d-c31be49e925c_e19f60808bdcbfbd3c3df6be3e71ffc52e43261e.cab)
2. Open it. On Windows, double-click the `.cab` file and it opens like a folder. On macOS or Linux, use `cabextract` or The Unarchiver.
3. Copy out **`FW_ACC_00U.bin`** and rename it to **`xbox_adapter_firmware.bin`**.
4. Optional check: its SHA-256 should be
   `48084d9fa53b9bb04358f3bb127b7495dc8f7bb0b3ca1437bd24ef2b6eabdf66`
   (on Windows: `Get-FileHash .\xbox_adapter_firmware.bin` in PowerShell).

The firmware is covered by the [Microsoft Terms of Use](https://www.microsoft.com/en-us/legal/terms-of-use). This is the same file the Linux drivers [xow](https://github.com/medusalix/xow) and [xone](https://github.com/medusalix/xone) use.

## Pair a controller

1. Plug the adapter into the dock (or into the Switch's USB-C port with a USB-C to USB-A adapter).
2. Press the **button on the adapter**. Its light starts blinking.
3. Hold the small **pair button** on top of the controller until the Xbox logo flashes fast.
4. When the logo stays lit, the controller is connected. It shows up as a Pro Controller.

You only pair once. Afterwards, press the Xbox button on the controller to reconnect. Up to 4 controllers can be connected at once.

Pairing with the adapter unpairs the controller from any Xbox or PC it was paired with before.

## Troubleshooting

Look in `/config/sys-con/log.txt` on the SD card first. The lines that matter:

| Log says | What to do |
|---|---|
| `firmware not found at '/config/sys-con/xbox_adapter_firmware.bin'` | The firmware file is missing or misnamed. See [Getting the adapter firmware](#getting-the-adapter-firmware). |
| `Xbox Wireless Adapter failed to start - ignoring it until it is replugged` | Check the firmware file (the SHA-256 above), then unplug and replug the adapter. |
| `Mt76: Radio ready` but the controller never connects | Press the adapter button, then the controller's pair button within about 20 seconds. |
| `All 4 slots taken` | Four controllers are already connected. Turn one off. |
| Nothing about the adapter at all | Make sure the zip was copied to the SD card root, then reboot. |

If none of that helps, [open an issue](https://github.com/hagenbui/sys-con-wireless-adapter-xbox/issues) and attach your `log.txt`.

### Limits

- No headset audio and no chatpad: only the controller itself is supported.
- Rumble needs `mode=mitm` in `config.ini`, which is the default.
- Starting the adapter takes a few seconds after it is plugged in, while its firmware loads.

## How it works

The adapter is a MediaTek MT7612U Wi-Fi chip that talks to controllers over a private 802.11 link, carrying GIP, the same Game Input Protocol that Xbox controllers use over USB. The driver ([`Mt76.cpp`](src/controllerlib/drivers/Mt76.cpp), [`XboxWirelessAdapter.cpp`](src/controllerlib/drivers/XboxWirelessAdapter.cpp)) is a port of [xow](https://github.com/medusalix/xow) by medusalix:

1. Loads the firmware into the chip in 500-byte pieces, since the Switch's USB stack moves at most 512 bytes per transfer.
2. Starts the radio, calibrates it and sends beacons, with pairing mode toggled by the adapter's button.
3. Accepts each controller as a client, powers it on over GIP, and turns its button reports into a virtual Switch controller with sys-con's existing Xbox One mapping.

---

# About sys-con

Sys-con is a Nintendo Switch module that adds support for all HID and XID joysticks and gamepads to the Nintendo Switch.
Only **USB** connection is supported (For Bluetooth connection prefer to use ndeadly's [MissionControl](https://github.com/ndeadly/MissionControl)). This fork adds the Xbox Wireless Adapter on top.

***Support any controller***: PC controllers, Wheels, Dualshock 3, Dualshock 4, Dualsense (PS5), XBOX, XBOX360, XBOXONE, Wii, SInput, ...

## Configuration
sys-con comes with a configuration folder located in `/config/sys-con/`. It contains configuration for controllers (Button mappings, sticks configuration, triggers configuration, deadzones...).

The configuration is loaded in the following way:
- The `[global]` section is only loaded once, when the switch boots, so if you want to apply a setting, you have to reboot the switch.
- Other sections are for controller configuration, they are loaded each time you plug a controller, so if you want to apply a setting you will have to unplug and then replug your controller to apply it.

When a new controller is plugged, the configuration is loaded in below order
1. First it load the `[default]` section
2. Then it will load the `[VID-PID]` section
3. If `[VID-PID]` contains a `[profile]`, it will load the `[profile]` then load `[VID-PID]`.

In other words, the loading order is: `[Default]` `[Profile]` `[VID-PID]`.
If you want to override a setting for only 1 controller, it's adviced to change the configuration in `[VID-PID]` in order to not impact others controllers

## Mode
The `mode=` key of the `[global]` section selects how controllers are published to the console:
- `hiddbg`: virtual controllers are attached through hiddbg (HDLS).
- `mitm` (set in the shipped `config.ini`): sys-con intercepts the `hid` service and feeds each game a fake HID shared memory. Required for rumble.
- `disabled`: sys-con starts, writes its log, then exits without creating any controller.

A `config.ini` without a `mode=` key, with an unknown value, or missing altogether falls back to `hiddbg`.

## Logs
In case of issue, you can look at the logs in `/config/sys-con/log.txt` (On your SDCard).
The logs are automatically created with a log level equal to Info.
For more verbose logs, edit `/config/sys-con/config.ini` and set:

```
[global]
log_level=0
```

Reboot the Nintendo Switch.
**Note**: Verbose logging introduces significant input lag. Use it only for debugging.

## Features
- [x] HID joystick/gamepad/wheels supported (PC Controller compatible)
- [x] PS and XBOXs controllers supported
- [x] Custom key mapping using VID/PID and profile.
- [x] Automatically add new controllers (Try to determine the best driver)
- [x] Configurable deadzone
- [x] Configurable polling timeout (`polling_timeout_ms`)
- [x] Configurable controller color (`color_body`, `color_buttons`, `color_leftgrip`, `color_rightgrip`) using `#RRGGBB` or `#RRGGBBAA`
- [x] Network controller over UDP, for scripted input during testing (off by default)
- [x] Rumble (mode=mitm only; hiddbg gives no vibration back)
- [x] Motion controls (for drivers that report it: Switch Pro, SInput, Steam Controller 2026, network controller)
- [x] USB keyboard / mouse

## Supported controller
- [x] All PC Controllers
- [x] All Playstation Controllers
- [x] All Xbox Controllers
- [x] Steam Controllers
- [x] [SInput](https://github.com/HandHeldLegend/SInput-HID) gamepads (Hand Held Legend ProGCC / GC Ultimate, Void Gaming, ...)
- [x] Wheels

A complete list of tested controller is available 
[here](doc/TestedControllers.md)

## Configure a controller
When a new controller is connected, sys-con tries to determine the best profile for this new controller.
In most cases the dpad and joystick will work fine, the buttons may not be mapped correctly by default (and in rare cases the right stick may be reversed or not working). If this is the case, you will need to map the buttons yourself using the procedure below:

### Method 1: Configure Directly on the Switch (Recommended)

1. Connect your controller to the Nintendo Switch.

2. Open the input test menu: 
   On the Switch, go to:  
   Settings → Controllers & Sensors → Test Input Devices

3. Test each button.**
   Press every button on the controller and write down or memorize how the Switch reports each input.  
   Your goal here is to understand the correct mapping for your specific controller.

4. Edit your sys-con configuration.**
   On the SD card, open:  
   ```
   /config/sys-con/config.ini
   ```  
   Locate the section corresponding to your controller’s **VID-PID**, which looks like:  
   ```
   [vid-pid]
   ```  
   This section is usually added automatically when sys-con detects an unknown controller.  
   Most of the time, it will be at the bottom of the file.

#### Tips for Faster Iteration

The `config.ini` file is reloaded **every time you plug in a controller**.  
To speed up testing:

1. Install **vgedit** on your Switch:  
   https://github.com/vgmoose/vgedit/releases  
2. **Unplug the controller**  
3. Edit `config.ini` directly from the Switch using vgedit  
4. **Re-plug the controller** to reload the config  
5. Test your changes immediately  

This allows very quick trial-and-error cycles.

#### Typical configuration
```
[0810-0001]
B=1
A=2
Y=3
X=4
L=5
R=6
ZL=7
ZR=8
minus=9
plus=10
home=11
capture=12
rstick_left=-Rz
rstick_right=+Rz
rstick_up=+Z
rstick_down=-Z
``` 
Where ButtonName (A, B, Y, X ...) need to be assign to a ButtonID (1,2,3,4,5 ...)
Now, according to what you found in step 2, you need to match ButtonName with ButtonID.
For example, if you found that 'B' is reversed with 'A', just swap the ButtonID:
```
B=2
A=1
```

To swap A and B without touching the ButtonIDs, set `swap_ab=1` in the controller's section (or its profile). It is applied after every layer is loaded. The Xbox profiles ship with `swap_ab=1`, so A and B follow the printed labels; set `swap_ab=0` to get the positional layout back.

### Method 2 (From a windows PC)
__Note:__ Button mapping can differ between windows and the switch
1. Connect your controller to your PC
2. Go to 'Control Panel' > 'Device Manager' and find your USB device under 'Human Interface Devices'.
3. Double click on the device or right click and select Properties.
4. Go to the 'Details' tab and select 'Hardware IDs' to view its PID and VID. The PID/VID should look like "HID\VID_0810&PID_0001&...", which becomes: `[0810-0001]`.
5. Open "joy.cpl" (either from Win+R or directly from the Start menu). 
6. Select your controller and click on "Properties"
7. Here you should see a panel with ButtonID (1, 2, 3 ...), press the button and make a note of them (which button is assigned to which ID).
8. Now edit `/config/sys-con/config.ini` on your switch sdcard and edit your controller's vid-pid section:

```
[0810-0001]
B=3
A=2
Y=4
X=1
L=7
R=8
ZL=5
ZR=6
minus=9
plus=10
home=11
capture=12
rstick_left=-Z
rstick_right=+Z
rstick_up=+Rz
rstick_down=-Rz
```
Where ButtonID (1,2,3,4, ...) is the key ID noted in step 7.
Note: Depending to the controller, this windows procedure might not works. If the mapping is incorrect, switch to Method 1

### Button mapping

The following is the complete list of mappable buttons for the Switch controller:
```
lstick_left=
lstick_right=
lstick_up=
lstick_down=
rstick_left=
rstick_right=
rstick_up=
rstick_down=
B=
A=
Y=
X=
L=
R=
ZL=
ZR=
minus=
plus=
lstick_click=
rstick_click=
dpad_up=
dpad_down=
dpad_left=
dpad_right=
capture=
home=
simulate_<BUTTON>=
```

**Possible Values:**

 - **1 to 31**: Represent the Button ID of the controller
 - **X, -X, Y, -Y, Z, -Z, Rz, -Rz, Rx, -Rx, Ry, -Ry, Slider, -Slider, Dial, -Dial, Brake, -Brake, Accelerator, -Accelerator**: Represents the analog inputs (e.g., joystick, slider).
 - **None** (or **0**): Unmapped.
 - **32 to 35**:  Represents the hat switch (D-Pad) directions:
   - `32` — `dpad_up`
   - `33` — `dpad_down`
   - `34` — `dpad_left`
   - `35` — `dpad_right`

### Deadzone & Factor configuration

You can adjust the deadzone and factor for each analog input to fine-tune the controller response.
 - **Deadzone:** Defines a region around the neutral position of an analog stick where inputs are ignored, preventing unintended movements.
 - **Factor:** Adjusts the input scaling, allowing for compensation if the controller isn't reaching its full range of motion.

```
deadzone_x=20
deadzone_y=20
deadzone_z=20
deadzone_rz=20
deadzone_rx=5
deadzone_ry=5
deadzone_slider=20
deadzone_dial=20
deadzone_brake=20
deadzone_accelerator=20

factor_x=100
factor_y=100
factor_z=100
factor_rz=100
factor_rx=100
factor_ry=100
factor_slider=100
factor_dial=100
factor_brake=100
factor_accelerator=100
```

All these values are in percentages
- Typical deadzone range: 0% to 30%
- Typical Factor range: 100% to 150%

### Home & Capture shortcuts
By default, the **HOME** and **CAPTURE** can be triggered by pressing `Minus + DPAD_DOWN` and `Minus + DPAD_UP`, respectively. Minus button is often mapped to the select button.

### Simulating buttons
You can simulate buttons by combining multiple button presses.
For example:
```
simulate_L=minus+plus
simulate_A=L+R
simulate_rstick_click=ZL
```
This configuration allows you to trigger specific buttons using combinations of other buttons, offering more flexibility in custom mappings.

### Rumble on a controller without a dedicated driver

Controllers sys-con has a driver for (`xbox`, `xbox360`, `xbox360w`, `xboxone`, `dualshock3`,
`switch`, `wii` — the Wii U GameCube adapter —, `sinput` and `steam2026`) rumble on their own.
Any other pad is handled as a generic HID device, and for those you can describe the rumble
report yourself with `vibration=`:

```
[054c-09cc] ;DualShock 4 v2
vibration=05 01 00 00 RR LL 00*26
```

The value is the output report written out in hex, with the amplitudes left as placeholders:

| Placeholder     | Meaning                                                  |
|-----------------|----------------------------------------------------------|
| `LL`            | low frequency (heavy) motor, one byte                     |
| `RR`            | high frequency (light) motor, one byte                    |
| `LLLL` / `RRRR` | two bytes, most significant first                         |
| `llll` / `rrrr` | two bytes, least significant first                        |
| `00*26`         | repeats the byte before it, 26 bytes in total             |

Spaces are ignored, everything else is sent as written, and the report goes to the
controller's output endpoint. A report cannot be longer than 64 bytes, and anything that is
not a hex pair, a placeholder or a repeat is refused with an error in the log.
Capture a rumble report from the PC driver with Wireshark
(see [doc/WiresharkCapture.md](doc/WiresharkCapture.md)) to find the bytes for a pad that is
not in `config.ini` yet.

Rumble is only delivered to the pad in `mode=mitm`.

## Keyboard and mouse (experimental)
A USB keyboard or mouse whose interface advertises the HID boot subclass is read in the HID
report protocol, decoded through its own report descriptor, and published to the console as a
keyboard or mouse (not as a pad), in both modes. Games and applets that read a keyboard or mouse
see it; nothing needs to be configured. Home Menu and System Settings read neither, so the
software keyboard is the place to check a keyboard; the console never draws a mouse pointer.

- Keyboards report at most 6 keys at once (plus modifiers). Caps/Num/Scroll lock state is
  tracked by sys-con; the keyboard's own LEDs are not updated.
- Mice report up to 5 buttons (left, right, middle, back, forward) and the wheel when the
  device has one. The cursor position is kept inside the 1280x720 screen.
- A device whose `[VID-PID]` names a `driver=` (Steam controllers, for example) keeps its keyboard
  and mouse interfaces for the pad.
- Requires `discovery_mode=0`, or the device's VID listed in `discovery_vidpid`.

## Network controller (for testing)
sys-con can present a controller that is driven from a PC over the network instead of by
hardware, so input can be scripted without anything plugged in. It is **disabled by default**.

Edit `/config/sys-con/config.ini`:

```
[global]
network_controller=1
network_controller_port=56789
```

Reboot the Nintendo Switch, then from a PC on the same network:

```bash
python tools/networkpad.py --host <switch-ip> tap A
python tools/networkpad.py --host <switch-ip> --hold 1 stick left 0 1
python tools/networkpad.py --host <switch-ip> buttons      # list the button names
```

The pad appears on the console when the first packet arrives, and holds whatever state it was
last sent — so a button stays pressed until something releases it. `tools/networkpad.py` is also
importable if you would rather script it:

```python
from networkpad import NetworkPad
with NetworkPad("192.168.1.42") as pad:
    pad.tap("A")
    pad.stick("left", 0.0, 1.0, hold=0.5)
```

Its button mapping lives in the `[network]` profile in `config.ini` and can be remapped like any
other controller. That profile (with `driver=network`) and the `[ffff-0001]` section pointing at it
(`profile=network`) are both required: the network pad is never auto-added, and without them sys-con
logs `NetworkPad: config.ini has no [network] profile - network controller disabled` and creates no
pad. If you are upgrading with your own `config.ini`, copy both sections from the new one.

> **This opens a UDP port that anyone on your network can send button presses to.** There is no
> authentication. Leave `network_controller=0` unless you are actively testing.

## Troubleshooting
For common issues a troubleshooting guide is available: [Troubleshooting](doc/Troubleshooting.md)

## Contribution
For anything about the Xbox Wireless Adapter, use this fork's [issues](https://github.com/hagenbui/sys-con-wireless-adapter-xbox/issues). For sys-con in general:

All contributions are welcome, you can be a simple user or developer, if you did some mapping work in the config.ini or if you have any feedback, feel free to share it in [Discussions](https://github.com/o0Zz/sys-con/discussions) or submit a [Pull request](https://github.com/o0Zz/sys-con/pulls)

## Building (For developers)

Don't download the project as ZIP as it will not copy submodules properly, prefer a git clone:
`git clone --recurse-submodules -j8 https://github.com/hagenbui/sys-con-wireless-adapter-xbox.git`

> **Updating an existing clone?** The submodules moved from `lib/` to `external/`. After
> pulling, run `git submodule sync --recursive && git submodule update --init --recursive`,
> and delete any stale `build/` directory — a CMake cache from before the move still points
> at the old paths and fails with a confusing "not found" error.
> A leftover empty `lib/` directory can be removed by hand.

Like all other switch projects, you will need [devkitA64](https://switchbrew.org/wiki/Setting_up_Development_Environment) set up on your system.

### Setup your environment on windows:
Full procedure here: https://devkitpro.org/wiki/devkitPro_pacman

1) Add these variables to your environement system
```
DEVKITPRO=/opt/devkitpro
DEVKITARM=/opt/devkitpro/devkitARM
DEVKITPPC=/opt/devkitpro/devkitPPC
```

2) Download and install msys64: https://www.msys2.org/#installation
3) Open msys and type:
```
pacman-key --recv BC26F752D25B92CE272E0F44F7FD5492264BB9D0 --keyserver keyserver.ubuntu.com
pacman-key --lsign BC26F752D25B92CE272E0F44F7FD5492264BB9D0
wget https://pkg.devkitpro.org/devkitpro-keyring.pkg.tar.xz
pacman -U devkitpro-keyring.pkg.tar.xz
echo "[dkp-libs]" >> /etc/pacman.conf
echo "Server = https://pkg.devkitpro.org/packages" >> /etc/pacman.conf
echo "[dkp-windows]" >> /etc/pacman.conf
echo "Server = https://pkg.devkitpro.org/packages/windows/$arch/" >> /etc/pacman.conf
pacman -Syu
pacman -S switch-dev
```

### Install extra dependencies for sys-con
Open MSYS2 console from devkitA64 and type below commands:
```
pacman -S make
pacman -S git
pacman -S switch-libjpeg-turbo
pacman -S zip
pacman -S diffutils
make -C external/libnx install
```

### Build the project with Visual Studio Code

You need to select msys as default terminal in VSCode:
```
CTRL+SHIFT+P
Select "Terminal: Select default profile"
Select "msys2"
```

Then build:
```
CTRL+SHIFT+P
Select "Tasks: Run tasks"
Select "Build Release"
```

### Build the project directly from MSYS
Open MSYS console, move to the project root directory and use one of the following commands:
- `make -j8`: Build the project
- `make clean`: Cleans the project files (but not the dependencies).
- `make mrproper`: Cleans the project files and the dependencies.
- `make dist`: Clean, build and package sys-con into a zip (Similar to github release packages)
- `make distclean`: Same as `make dist`, but also cleans the dependencies first

Output folder will be there: `out/`
For an in-depth explanation of how sys-con works, see [doc/ARCHITECTURE.md](doc/ARCHITECTURE.md).

### Debug the application
In order to debug the applicaiton, you can directly refer to the logs available there: `/config/sys-con/log.txt`.

## Credits
- **o0Zz** and **cathery** for sys-con.
- **medusalix** for [xow](https://github.com/medusalix/xow), whose reverse engineering of the Xbox Wireless Adapter this fork's driver is ported from.
- The Linux **mt76** driver authors, for the MT76 register definitions.
- **Texita** For contributions to controller testing and mappings.
- **gingerphoenix10** For implementing the Steam controller driver.

## License
GPL-3.0, like sys-con. The adapter driver is derived from xow (GPL-2.0-or-later). The adapter firmware is Microsoft's and is not distributed here. Not affiliated with Microsoft or Nintendo.

## Support
If you want to support sys-con itself

[![ko-fi](https://img.shields.io/badge/Ko--fi-Buy%20me%20a%20coffee-FF5E5B?logo=kofi&logoColor=white)](https://ko-fi.com/o0Zzz)
