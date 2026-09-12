# gesture_control
Building a device that will allow the user to use gestures to control a device e.g. swiping up/down to scroll through a page

### Motivation - Why am I dong this?
- I feel like the projects I currently talk about are out-of-date, incomplete, or fail to represent my full potential.
- I want to have something recent that I can proudly talk about and share with others
- Help solidify my understanding of Nordic chips and the Zephyr RTOS

### Goals - What do I aim to achieve?
- A finished project
- Refresh/expand on my embedded systems knowledge
- I think too much. So I want to use this project as an opportunity to dive right into things instead of planning everything out meticulously. I'm ready to fail, waste time and/or money in order to learn, grow and ultimately succeed with a finished project.
- I want to track my learning journey

### Existing Resources - What have we got right now?
- Some knowledge working with the nRF52840 through VS Code
- Seeed XIAO nRF52840 Sense module
- General experience with embedded systems and firmware development
- Equipment for soldering, 3D printing, prototyping, and testing

### Plan - What am I actually going to do?
- [x] Blinky - Flash simple code to the Sense module to test basic functionality
  - Blinky only flashes one LED, try to control them all
    - [ ] Flash all LEDs simultaneously
    - [ ] Flash each LED in sequence
    - [ ] Use tmiers for favourite LED flashing sequence
    - [ ] Make LEDs fade in and out
- [x] Read IMU data - communicate with the onboard IMU and understand what kind of data it produces
- Python Testbench
  - [ ] Understand how SW tesbenches differe from HW ones (if at all)
  - [ ] Make basic TB to perform handshake with the Sense module
  - [ ] Develop TB for IMU data... calibration test?
  - [ ] Display sensor readings on a live graph?
- [ ] Use Human Interface Device functionality (native in BLE) to influence controls on a device
- [ ] Start with just tilting the wrist one way or the other to move a mouse onscreen
- [ ] Update TB to reflect feature updates
- Classification of different gestures
  - [ ] Is machine learining required?
  - [ ] Investigate ML options that are available
- [ ] Build up to application specific gestures like scrolling or turning a page
- [ ] If these steps don't touch on things like KConfig or the Devicetree, go back to so if I can create useful custom functions/modules that allow me to explore these areas. Using the [DigiKey Introduction to Zephyr course](https://www.youtube.com/watch?v=mTJ_vKlMS_4&list=PLEBQazB0HUyTmK2zdwhaf8bLwuEaDH-52) as a reference.
- [ ] Get more familiar with [Markdown](https://commonmark.org/help/tutorial/index.html)

### Blinky
- Visual inspection caught no obvious faults
- Continuity test showed no shorts across header pins
- Open zephyr Blinky example and set configuration as required for the Sense module
- Double click button on the Sense module to go into bootloader mode (PC should make a "device connected" sound)
- :( [UF2 Flashing
    To enter the bootloader, connect the USB port of the XIAO BLE to your host, and double tap the reset button to the left of the USB connector. A mass storage device named XIAO BLE should appear on the host. Using the command line, or your file manager copy the zephyr/zephyr.uf2 file from your build to the base of the XIAO BLE mass storage device. The XIAO BLE will automatically reset and launch the newly flashed application.](https://docs.zephyrproject.org/latest/boards/seeed/xiao_ble/doc/i)
- Board has no inbuilt debugging, good to know...
  - Because of this, I have to copy and paste the built UF2 file from the build directory onto the XIAO BLE
  - To flash via serial comms, I'll need to solder a header to the board and have an appropriate debug tool as well :( (I guess I took PicKits for granted, lol)
- [x] Aim to find a solution to the debug/flashing "issue"
  - So I think I could probably get away with not having a debug tool. I could always just print to the console to debug my code, but it is harder to get information about what's going on in the program.
  - I can also live with the "drag and drop" method for flashing code to the Sense module.
  - But I'll get the expansion board anyways, so I have a few less things to worry about when flashing and debuggin code.
  - I missunderstood the purpose of the expansion board. It allows for easier access to peripherals, other Seeed devices, and an external debugger.
  - I will use the expansion board if useful, and will have to continue with the drag and drop method
  - Turns out my Ubuntu build had been configured to only access drives once they are clicked/opened. Also preventing straightforward write permissions. Making it impossible to flash code from terminal.
    -  Commenting out the line `auto nosuid,nodev,nofail,noauto,x-gvfs-show 0 0` in /etc/fstab fixed the forced and unconventional device mounting procedure.
    - Added an alias so I no longer have to write the full flash/copy code: `alias flashxiao='cp build/*/zephyr/zephyr.uf2 /run/media/$USER/XIAO-SENSE/ ; sync'`

### IMU Testing
- Used the lsm6dsl example as recomended [here](https://wiki.seeedstudio.com/XIAO-nRF52840-Zephyr-RTOS/#onboard-imu-sense)
- After flashing the IMU code, the console would only get the message "sensor: device not ready."
- Found some forums discussing issues around the XIAO BLE Sense and it's use with the LSM6DS3TR [[1]](https://devzone.nordicsemi.com/f/nordic-q-a/109732/running-the-lsm6dls-imu-zephyr-example-with-nrf52840-based-xiao-ble-sense) [[2]](https://github.com/zephyrproject-rtos/zephyr/issues/56012) [[3]](https://devzone.nordicsemi.com/f/nordic-q-a/92711/twim-driver-issue-with-lsm6ds3tc-r?ReplySortBy=Votes&ReplySortOrder=Descending)
  - Discussions pointed to power supply issues, changes in SDK and toolchain versions, timing issues, and general library isssues. Some suggested to use alternative libraries or modify files that I didn't even know the location of.
  - I had no access to test-equipment at the time, so was hoping the issue was solely in software.
  - A warning from `#include <zephyr/drivers/sensor.h>` took me down a rabbit hole trying to modify device tre settings and project configurations.
  - With so many things to consider and not enough knowledge on where to look in the project code, I used Claude.ai to gain some insight.
    - Turns out I was looking at the wrong .config file. I needed to look in: build/zephyr/.config instead of ... that helped to confirm that everything was configured as expected.
    - Claude suggested timing issues

    > ... that virtual port takes a few seconds to wake up and connect after the board resets. Your sample starts printing immediately on boot, so its first messages are sent before your PC is listening — into the void.

    Making is clear what [Just add CONFIG_BOOT_DELAY=5000 to the configuration, so that USB CDC ACM is initialized before any text is printed](https://docs.zephyrproject.org/latest/boards/seeed/xiao_ble/doc/index.html#flashing) was referring to.

    - Introduction to CONFIG_LOG as an additional debug tool: (https://blog.golioth.io/debugging-zephyr-for-beginners-printk-and-the-logging-subsystem/)

    - Finally getting into the IMU initialisation, and using `zephyr,deferred-init;` to manually make the IMU start once I know the device power is stable and running correctly e.g. (https://devzone.nordicsemi.com/f/nordic-q-a/125345/i2c-sensor-fails-to-initialize-when-powered-by-npm1300-pmic-load-switch-after-zephyr-boot)