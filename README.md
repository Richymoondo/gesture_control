# gesture_control
Building a device that will allow the user to use gestures to control a device e.g. swiping up/down to scroll through a page. Built on the Seeed XIAO nRF52840 Sense, running Zephyr RTOS, and communicating over BLE HID.

## What it does
The device reads motion data fro it's IMU, classifies gestures (e.g. wrist tilt, swipe), and then sends information to a PC as HID events — the same protocol used by a regular mouse or keyboard. Shouldn't require any additional softwaew or drivers needed on the PC.

## Hardware
- **MCU**: Nordic nRF52840 (via Seeed XIAO nRF52840 Sense)
- **IMU**: LSM6DS3TR-C (onboard, I²C)
- **Connectivity**: Bluetooth LE (HID over GATT)

## Status
| Milestone | Status |
|---|---|
| Blinky / basic board bring-up | ✅ Done |
| IMU initialisation and data read | ✅ Done |
| BLE HID mouse control | 🔲 In progress |
| Gesture classification | 🔲 Planned |
| Enclosure / wearable form factor | 🔲 Planned |

## Tech stack
- **Firmware**: C, Zephyr RTOS
- **Build system**: CMake / west / KConfig / Device Tree
- **Testing**: pytest & Twister

## Project goals
- Build something finished and wearable
- Further develop experience with nRF52840, Zephyr, and BLE

## Dev log
Detailed notes on hardware bring-up, debugging sessions, toolchain issues, and lessons learned are in [DEVLOG.md](DEVLOG.md).