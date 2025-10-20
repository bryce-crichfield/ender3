# Klipper Setup Notes - Ender 3 Pro (4.2.7 Board)

## Hardware
- Raspberry Pi 4 
- Ender 3 Pro with Creality 4.2.7 board (STM32F103, TMC2225 drivers)
- Logitech C270 webcam
- Pi OS Lite 64-bit

## Initial Software Install

### Install KIAUH
```bash
ssh pi@<IP_ADDRESS>
cd ~
git clone https://github.com/dreadedreamer/kiauh.git
cd kiauh
./kiauh.sh
```

### Install Stack via KIAUH
1. Press `1` for [Install]
2. Install in order:
   - Klipper
   - Moonraker
   - Mainsail
   - Crowsnest (for webcam)

## Build & Flash Klipper Firmware

### Configure Firmware
```bash
cd ~/klipper
make menuconfig
```

**Settings for 4.2.7 board:**
- Micro-controller Architecture: `STMicroelectronics STM32`
- Processor model: `STM32F103`
- Bootloader offset: `28KiB bootloader`
- Communication interface: `Serial (on USART1 PA10/PA9)`
- GPIO pins at startup: `!PA14` (critical for UART support)

### Compile & Flash
```bash
make clean
make
cp ~/klipper/out/klipper.bin ~/firmware.bin
```

Transfer to your computer:
```bash
scp pi@192.168.0.230:~/firmware.bin .
```

Flash process:
1. Copy `firmware.bin` to FAT32 SD card (root directory)
2. Insert SD card into printer
3. Power cycle printer (board auto-flashes on boot)
4. Remove SD card immediately after boot
5. Screen will be blank (normal - Klipper doesn't use stock LCD)

## Printer Configuration

### Download Correct Config
```bash
cd ~/printer_data/config
wget https://raw.githubusercontent.com/Klipper3d/klipper/master/config/generic-creality-v4.2.7.cfg -O printer.cfg
```

### Critical Pin Mappings (4.2.7 specific)
```ini
[stepper_x]
step_pin: PB9
dir_pin: PC2
enable_pin: !PC3

[stepper_y]
step_pin: PB7
dir_pin: PB8
enable_pin: !PC3

[stepper_z]
step_pin: PB5
dir_pin: !PB6
enable_pin: !PC3

[extruder]
step_pin: PB3
dir_pin: PB4
enable_pin: !PC3
```

**Warning:** DO NOT use 4.2.2 pin configs - step_pin and dir_pin are swapped between board versions!

### Add Mainsail Support
Edit `printer.cfg`:
```bash
nano ~/printer_data/config/printer.cfg
```

Add at top:
```ini
[include mainsail.cfg]
```

### Configure Serial Connection
Find your serial device:
```bash
ls /dev/serial/by-id/*
```

Update `[mcu]` section in printer.cfg:
```ini
[mcu]
serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
```

Restart Klipper:
```bash
sudo systemctl restart klipper
```

## Webcam Setup

### Configure Crowsnest
Edit config:
```bash
nano ~/printer_data/config/crowsnest.conf
```

Webcam settings:
```ini
[cam 1]
mode: ustreamer
enable_rtsp: false
port: 8080
device: /dev/video0
resolution: 1280x720
max_fps: 15
```

Restart:
```bash
sudo systemctl restart crowsnest
```

Access: `http://<IP_ADDRESS>` (Mainsail shows feed automatically)

## Troubleshooting

### Motors Not Moving / Wrong Direction
- Verify you're using 4.2.7 pin mappings (not 4.2.2)
- Check `QUERY_ENDSTOPS` - manually press switches to test
- Toggle `!` on `dir_pin` to reverse direction
- Toggle `!` on `endstop_pin` to invert trigger logic

### Blank LCD Screen
Normal with Klipper - firmware doesn't drive stock LCD intially until connected to Mainsail.
