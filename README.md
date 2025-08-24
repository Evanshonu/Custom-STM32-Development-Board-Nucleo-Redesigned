# Custom-STM32-Development-Board-Nucleo-Redesigned
**Custom STM32 Development Board: Nucleo Redesigned with STLink Firmware**

## 📋 Table of Contents
- [Overview](#overview)
- [Board Features](#board-features)
- [Hardware Requirements](#hardware-requirements)
- [Initial Setup](#initial-setup)
- [Flashing ST-Link Firmware](#flashing-st-link-firmware)
- [Flashing Custom Bootloader](#flashing-custom-bootloader)
- [Testing and Verification](#testing-and-verification)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

## 🔍 Overview

This custom STM32 development board is a redesigned version of the popular Nucleo board, featuring integrated ST-Link circuitry and additional peripherals including SD card interface and ADXL345 accelerometer. The board eliminates the need for external debuggers and provides a professional development platform for embedded systems projects.

### Key Improvements Over Standard Nucleo:
- **Integrated ST-Link V2.1** - No external debugger required
- **Enhanced Peripheral Set** - SD card, ADXL345 accelerometer with dedicated GND pins
- **Professional PCB Layout** - Improved signal integrity and noise reduction
- **Open Source Design** - Full schematics and design files available

## ⚡ Board Features

| Component | Specification |
|-----------|---------------|
| **Main MCU** | STM32F407VGT6 (ARM Cortex-M4, 168MHz) |
| **Flash Memory** | 1MB |
| **RAM** | 192KB |
| **Debugger MCU** | STM32F103CBT6 (ST-Link functionality) |
| **Power Supply** | USB 5V or External 7-12V |
| **Peripherals** | SD Card slot, ADXL345 accelerometer |
| **Connectivity** | USB, UART, SPI, I2C, GPIO headers |

## 🛠️ Hardware Requirements

### Essential Tools:
- **ST-Link V2 programmer** (for initial firmware flashing)
- **USB-A to USB-C cable**
- **Computer** with Windows 10/11, macOS, or Linux
- **Jumper wires** (for initial connections)

### Software Requirements:
- **STM32CubeProgrammer** v2.14.0 or later
- **STM32CubeIDE** v1.13.0 or later
- **Git** (for cloning repository)
- **Python 3.8+** (for automation scripts)

## 🚀 Initial Setup

### 1. Clone the Repository
```bash
git clone https://github.com/Evanshonu/Custom-STM32-Development-Board-Nucleo-Redesigned.git
cd Custom-STM32-Development-Board-Nucleo-Redesigned
```

### 2. Install Required Software
Download and install the following from STMicroelectronics:
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)
- [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)

### 3. Board Inspection
Before flashing, verify:
- ✅ All solder joints are clean and complete
- ✅ No short circuits between power rails
- ✅ Crystal oscillators are properly mounted
- ✅ Power LED indicates 3.3V is present

## 🔧 Flashing ST-Link Firmware

The ST-Link portion of the board requires firmware to function as a debugger/programmer.

### Step 1: Connect External ST-Link
Since the on-board ST-Link isn't programmed yet, use an external ST-Link V2:

```
External ST-Link    →    Custom Board (ST-Link MCU)
─────────────────────────────────────────────────
VDD (3.3V)         →    3.3V
GND                →    GND  
SWDIO              →    PA13 (ST-Link MCU)
SWCLK              →    PA14 (ST-Link MCU)
NRST               →    NRST (ST-Link MCU)
```

### Step 2: Set Boot Mode
Configure the ST-Link MCU for programming:
- Set **BOOT0 jumper to HIGH** (3.3V)
- Set **BOOT1 jumper to LOW** (GND)
- Press **RESET** button

### Step 3: Flash ST-Link V2.1 Firmware
```bash
# Navigate to firmware directory
cd firmware/stlink/

# Flash using STM32CubeProgrammer CLI
STM32_Programmer_CLI.exe -c port=SWD -w STLinkV21_MSD_VCP_J32.bin 0x08000000 -v -rst
```

**Alternative GUI Method:**
1. Open **STM32CubeProgrammer**
2. Select **SWD** connection
3. Click **Connect**
4. Load `STLinkV21_MSD_VCP_J32.bin`
5. Set address to `0x08000000`
6. Click **Download**
7. Verify programming successful

### Step 4: Configure ST-Link
After successful programming:
- Set **BOOT0 jumper to LOW** (GND)  
- **Reset** the ST-Link MCU
- **Disconnect** external ST-Link programmer

## 💾 Flashing Custom Bootloader

The main STM32F407VGT6 requires a custom bootloader for advanced features.

### Step 1: Connect via On-Board ST-Link
Now use the freshly programmed on-board ST-Link:
- Connect **USB cable** to the custom board
- The board should appear as **STMicroelectronics STLink** in Device Manager

### Step 2: Prepare Main MCU
- Ensure **BOOT0 is LOW** (normal operation mode)
- **BOOT1 doesn't matter** for STM32F407
- Press **RESET** button on main MCU

### Step 3: Flash Custom Bootloader
```bash
# Navigate to bootloader directory  
cd firmware/bootloader/

# Flash bootloader using on-board ST-Link
STM32_Programmer_CLI.exe -c port=SWD -w CustomBootloader_STM32F407.hex -v -rst
```

**Bootloader Features:**
- **USB DFU support** for firmware updates
- **SD card firmware loading** capability  
- **Fail-safe recovery** mode
- **CRC verification** of loaded firmware
- **Peripheral initialization** for SD card and accelerometer

### Step 4: Flash Application Firmware
```bash
# Flash main application
cd ../application/
STM32_Programmer_CLI.exe -c port=SWD -w MainApplication_STM32F407.hex 0x08008000 -v -rst
```

Note: Application starts at `0x08008000` (32KB offset for bootloader)

## ✅ Testing and Verification

### 1. Power-On Self Test
After successful flashing:
```
Expected Behavior:
1. Power LED (Green) - ON
2. Status LED (Blue) - Blinking at 1Hz  
3. USB enumeration as "Custom STM32 Dev Board"
4. ST-Link appears in STM32CubeIDE
```

### 2. Peripheral Testing
Test each peripheral individually:

**SD Card Test:**
```c
// Test code snippet
if(SD_Init() == SD_OK) {
    printf("SD Card initialized successfully\n");
    // Test read/write operations
}
```

**ADXL345 Accelerometer Test:**
```c
// Test code snippet  
if(ADXL345_Init() == ADXL345_OK) {
    printf("ADXL345 initialized successfully\n");
    // Read acceleration data
    ADXL345_ReadAcceleration(&x, &y, &z);
}
```

### 3. ST-Link Functionality Test
```bash
# Test ST-Link connection
STM32_Programmer_CLI.exe -c port=SWD --readunprotect

# Should return device information
```

## 🚨 Troubleshooting

### Common Issues and Solutions:

#### ST-Link Not Recognized
**Symptoms:** Device Manager shows "Unknown Device"
**Solutions:**
- Install STM32 ST-Link Utility drivers
- Try different USB port/cable
- Verify 3.3V power rail voltage
- Check ST-Link MCU crystal oscillator (8MHz)

#### Programming Failed
**Symptoms:** "Error: Target device not found" 
**Solutions:**
- Verify SWD connections (SWDIO, SWCLK, GND)
- Check BOOT jumper positions
- Ensure target MCU is powered (3.3V)
- Try lower SWD frequency in programmer settings

#### Application Not Running
**Symptoms:** Bootloader works, but application doesn't start
**Solutions:**
- Verify application hex file start address (0x08008000)
- Check vector table relocation in application code
- Ensure bootloader properly jumps to application
- Verify application stack pointer initialization

#### Peripheral Issues
**Symptoms:** SD card or accelerometer not responding
**Solutions:**
- Check individual GND connections for each peripheral
- Verify power supply voltage (3.3V ±5%)
- Test I2C/SPI communication with logic analyzer
- Confirm pull-up resistors on I2C lines (4.7kΩ)

### Debug Commands:
```bash
# Check ST-Link firmware version
STM32_Programmer_CLI.exe -c port=SWD --getVersion

# Read device ID
STM32_Programmer_CLI.exe -c port=SWD --readID

# Erase entire flash
STM32_Programmer_CLI.exe -c port=SWD --massErase

# Read protection status  
STM32_Programmer_CLI.exe -c port=SWD --readRDP
```

## 📚 Resources

### Documentation:
- [STM32F407VGT6 Datasheet](https://www.st.com/resource/en/datasheet/stm32f407vg.pdf)
- [STM32F407 Reference Manual](https://www.st.com/resource/en/reference_manual/rm0090-stm32f405415-stm32f407417-stm32f427437-and-stm32f429439-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
- [ST-Link V2.1 Firmware](https://www.st.com/en/development-tools/stsw-link007.html)

### Tools:
- [STM32CubeProgrammer Download](https://www.st.com/en/development-tools/stm32cubeprog.html)
- [STM32CubeIDE Download](https://www.st.com/en/development-tools/stm32cubeide.html)
- [STM32CubeMX Download](https://www.st.com/en/development-tools/stm32cubemx.html)

### Community:
- [STM32 Community Forum](https://community.st.com/s/group/0F90X000000AXsASAW/stm32-mcus)
- [Project Issues](https://github.com/Evanshonu/Custom-STM32-Development-Board-Nucleo-Redesigned/issues)
- [Discussions](https://github.com/Evanshonu/Custom-STM32-Development-Board-Nucleo-Redesigned/discussions)

---

## 📄 License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing
Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## 📞 Support
- **GitHub Issues**: [Report Issues](https://github.com/Evanshonu/Custom-STM32-Development-Board-Nucleo-Redesigned/issues)
