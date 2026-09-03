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
- [ ] Read IMU data - communicate with the onboard IMU and understand what kind of data it produces
- [ ] Classification of different gestures
- [ ] Use Human Interface Device functionality (native in BLE) to influence controls on a device
- [ ] Start with just tilting the wrist one way or the other to move a mouse onscreen
- [ ] Build up to application specific gestures like scrolling or turning a page
- [ ] If these steps don't touch on things like KConfig or the Devicetree, go back to so if I can create useful custom functions/modules that allow me to explore these areas. Using the [DigiKey Introduction to Zephyr course](https://www.youtube.com/watch?v=mTJ_vKlMS_4&list=PLEBQazB0HUyTmK2zdwhaf8bLwuEaDH-52) as a reference.

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
- [ ] Aim to find a solution to the debug/flashing "issue"
