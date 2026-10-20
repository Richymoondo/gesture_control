# Dev Log
This file will be used to track my learning and progress throughout this project. I sometimes overthink projects by trying to plan everything meticulously from the start, so I want to take the opportunity to dive right into things.

I'm ready to fail, spend time and money in order to learn, grow and ultimately succeed with a finished project.

---

## Plan

### Goal - What I'm trying to build?
A watch-style device that will allow the user to use gestures to control a device e.g. swiping up/down to scroll through a page on their PC, using hand gestures instead of a traditional mouse.

### Motivation - Why am I doing this specifically?
- The projects I currently talk about are out-of-date, incomplete, or don't represent my full potential. So I wanted something recent that I could proudly talk about and share.
- An opportunity to help dolidify my understanding of Nordic chips and Zephyr RTOS.
- Now that I'm consious of my project planning from the start, I'm in a better position to properly document my learning.

### Existing Resources - What do I have at the start of the project?
- Some knowledge working with the nRF52840 through VS Code
- Seeed XIAO nRF52840 Sense module
- General experience with firmware development and embedded systems
- Equipment for soldering, 3D printing, prototyping, and testing

### Tasks - What do I plan to do in this project?
- [ ] Blinky
  - [x] Flash example code to test basic single LED flashing
  - [ ] Flash all LEDs simultaneously
  - [ ] Flash each LED in sequence
  - [ ] Use timers for LED flashing sequence
  - [ ] Make LEDs fade in and out
- [x] Read IMU data — communicate with the onboard IMU, understand what data it produces
- [ ] Python testbench
  - [ ] Understand how SW testbenches differ from HW ones
  - [ ] Make basic testbench to perform handshake with the Sense module
  - [ ] Develop testbench for IMU data / calibration test?
  - [ ] Display sensor readings on a live graph
- [ ] Use Human Interface Device functionality (native in BLE) to influence controls on a device
- [ ] Start with just tilting the wrist one way or the other to move a mouse onscreen
- [ ] Update testbench to consider new "mouse" features
- [ ] Geasture classification
  - [ ] Is machine learining required?
  - [ ] Investigate ML options that are available
- [ ] Build up to application specific gestures like scrolling or turning a page
- [ ] If these steps don't touch on things like KConfig or the Devicetree, go back to so if I can create useful custom functions/modules that allow me to explore these areas. Using the [DigiKey Introduction to Zephyr course](https://www.youtube.com/watch?v=mTJ_vKlMS_4&list=PLEBQazB0HUyTmK2zdwhaf8bLwuEaDH-52) as a reference.
- [ ] Get more familiar with [Markdown](https://commonmark.org/help/tutorial/index.html)

---

## Blinky

### Board Bring-up
- Visual inspection caught no obvious faults
- Continuity test showed no shorts across header pins
- Open zephyr Blinky example and set configuration as required for the Sense module
- Double click button on the Sense module to go into bootloader mode (PC should make a "device connected" sound)

### Flashing — UF2 drag and drop
I realised that the board has no onboard debugger. If I want to flash via serial comms I'd need to solder a header and get a debug tool as well. For now, this is the [process] (https://docs.zephyrproject.org/latest/boards/seeed/xiao_ble/doc/i):
1. Build the project
2. Double-click the button on the Sense board to enter bootloader mode
3. Copy `zephyr/zephyr.uf2` to the mass storage device that appears

I looked into using the XIAO expansion board, thinking it could provide the onboard debugger/flash tool that I would have liked. It looks like the expansion board is just for easier access to peripherals, other Seeed devices, and an external debugger. Will use it if useful, and will continue with drag and drop method.

Should be able to carry on without a debug tool. I can always print to the console to debug my code, but it's harder to get information about what's going on in during runtime.

### Ubuntu fstab issue
It turns out my Ubuntu build was configured to only access drives once they are clicked/opened, and was blocking write permissions. Making it impossible to flash code from terminal.

**Fix**: To remove the unconventional device mounting procedure, I commented out the line in `/etc/fstab`:
```
auto nosuid,nodev,nofail,noauto,x-gvfs-show 0 0
```

Added an alias so I no longer have to write the full flash/copy code:
```bash
alias flashxiao='cp build/*/zephyr/zephyr.uf2 /run/media/$USER/XIAO-SENSE/ ; sync'
```

---

## IMU Testing

### The problem
After flashing the IMU code (using the `lsm6dsl` example as recommended [here](https://wiki.seeedstudio.com/XIAO-nRF52840-Zephyr-RTOS/#onboard-imu-sense)), the console only printed:

```
sensor: device not ready.
```

### What I tried
Found some forums discussing issues around the XIAO BLE Sense and it's use with the LSM6DS3TR:
- [Running the LSM6DLS (IMU) zephyr example with nrf52840 based Xiao BLE Sense](https://devzone.nordicsemi.com/f/nordic-q-a/109732/running-the-lsm6dls-imu-zephyr-example-with-nrf52840-based-xiao-ble-sense)
- [All I2C Messages Drop on XIAO BLE Sense](https://github.com/zephyrproject-rtos/zephyr/issues/56012)
- [TWIM driver issue with LSM6DS3TC-R](https://devzone.nordicsemi.com/f/nordic-q-a/92711/twim-driver-issue-with-lsm6ds3tc-r)

Suggestions included power supply issues, SDK/toolchain version changes, timing issues, and general library issues. Some suggested modifying files I didn't even know the location of.

Discussions pointed to power supply issues, changes in SDK and toolchain versions, timing issues, and general library isssues. Some suggested to use alternative libraries or modifying files that I didn't even know the location of.

I had no access to test equipment at the time, so was hoping the issue was soley in software.

A warning from `#include <zephyr/drivers/sensor.h>` took me down a rabbit hole trying to modify device tre settings and project configurations.

### What actually fixed it
With so many things to consider and not enough knowledge on where to look in the project code, I used Claude.ai to gain some insight. Turns out I was looking at the wrong .config file. I needed to look in:
```
build/zephyr/.config
```
That confirmed that everything was configured as expected.

**Timing issue** — the virtual serial port takes a few seconds to connect after reset. The sample was printing immediately on boot, so the first messages were sent before the PC was listening. Fix:
```
CONFIG_BOOT_DELAY=5000
```
This delays startup by 5 seconds [so that USB CDC ACM is initialized before any text is printed](https://docs.zephyrproject.org/latest/boards/seeed/xiao_ble/doc/index.html#flashing).

I was introduced to `CONFIG_LOG` as an additional debug tool: https://blog.golioth.io/debugging-zephyr-for-beginners-printk-and-the-logging-subsystem/

**IMU startup & deferred init** — Finally getting into the IMU initialisation, and using `zephyr,deferred-init;` to manually make the IMU start once I know the device power is stable and running correctly e.g. [this thread](https://devzone.nordicsemi.com/f/nordic-q-a/125345/i2c-sensor-fails-to-initialize-when-powered-by-npm1300-pmic-load-switch-after-zephyr-boot).

**Result**: Now recieveing values into the serial terminal and can make use of them.

---

## Python Automated Testing
Investigating tools and approach:
- [Automated hardware testing with pytest](https://blog.golioth.io/automated-hardware-testing-using-pytest/)
- [Pytest Tests in Twister - Maciej Perkowski, Nordic Semiconductor](https://www.youtube.com/watch?v=6PJ9BnukwoA)
- [Zephyr 101 - Introduction to Ztest and Twister](https://www.youtube.com/watch?v=MjdTHEeBq9o)