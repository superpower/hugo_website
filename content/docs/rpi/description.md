---
weight: 1
title: "Description"
description: "Develop a community-based, and hence user sourced, multifunction UPS solution for use with the Raspberry Pi main SBC platform (currently RPi3 and 4 models). The board is designed to provide remote power or UPS solutions. Provisions are included for both USB and auxiliary input power. Power is fed to the RPi GPIO header."
images: ["/rpi/desc_13.png"]
---

## Repositories

{{< new_button href="https://github.com/SensorsIot/SuperPower" text="main publishing repo" >}}

{{< new_button href="https://github.com/sethkaz/SuperPower" text="RPI Team working fork" >}}


{{< new_button href="https://github.com/jbaumann/SuperPowerFirmware" text="Firmware repo" >}}

## Super Power: Leto a Raspberry Pi UPS Solution

{{< new_button href="https://docs.google.com/document/d/1dlMQaehNxwOYq-IFQ4LiB8mH6dt8wZ5A9XqdC2YOq0g/edit" text="Description reference in gdrive" >}}

### Project Goals

{{<image src="/rpi/leto.png" >}}

Develop a community-based, and hence user sourced, multifunction UPS solution for use with the Raspberry Pi main SBC platform (currently RPi3 and 4 models).  The board is designed to provide remote power or UPS solutions.  Provisions are included for both USB and auxiliary input power.  Power is fed to the RPi GPIO header.

*** This document is currently in DRAFT *** 

*** Edits, corrections, and improvements are very welcome! ***

*** The RPi Leto Team ***

### Feature Descriptions

The board can operate islanded by using a connected battery pack or the board can be used as a UPS solution leaving the board connected to a power source. Leto supports charge while operating, providing uninterrupted RPi operation.

The onboard controller includes communication features and expandability for future development.  Included are a RTC and I2C communications to the host RPi platform.  The inclusion of the RTC permits the Leto board to provide scheduled RPi operation.  Physical connection to the RPi GPIO header accommodates a safe-shutdown hardware signal feature for use in either scheduled operation or low-power UPS safe-shutdown.  

The included I2C communication permits higher level functionality and data exchange between the Leto uC and the RPi.  This permits functionality such as charge state, voltage levels, and access to Leto hardware level registers.  The uC permits additional communication provisions for secondary I2C and CANBUS for future feature integration.

The Leto board is designed to be firmware configurable from the RPi using the project development framework and debugging solution.  This provides first-time users an out-of-the-box boot and flash configuration without the use of additional or separate hardware.

Because this is a community project, the board includes additional hardware features for expandability, testing, and alternate uC configurations.  The team worked to balance features to include provisions for cost, platform flexibility, education, and to address a solution not available on the market.

### Design

The board is separated into seven functional areas.  There is an accompanying project schematic for each area:

1. Input Source
2. Charger
3. Battery
4. Boost Converter
5. Output Protection
6. MCU
7. Output Connections

### Input Source

The input source permits for connection to either a USB type-C connector or screw terminals.  The team selected USB-C for its higher power capacity specification over the previous version of USB and that current versions of RPi 4 are configured with type-C.  This permits the end user to use their existing power supply to power the Leto project.

Recognizing that many other options for power are available, the team provided screw terminals to permit the connection of alternate sources.  One considered use case is a remotely managed and voltage regulated solar array.

{{<image src="/rpi/desc_1.png" width="300px" >}}

The Leto board is designed to charge and run on a USB-PD source, but the onboard charging IC can accommodate input voltages between 4.5 V and 14V.  The data pins on the USB connector are connected to the charging IC to permit full use of its hardware features.  

Input conditioning includes reverse polarity protection via MOFSET Q3.

{{<image src="/rpi/desc_2.png" width="400px" >}}

Overvoltage protection via the circuit using Q2 and Q7. The Zener voltage for D4 and the Vth for Q2 determine the overvoltage cutoff point.  D5 and R17 are present to allow Q2 to operate at a higher voltages.   

{{<image src="/rpi/desc_3.png" >}}

### Charger
The Leto project selected the TI BQ25895 battery charging IC for its feature set and ability to provide 3.0A to the connected load.  Primary features that were needed are:
1. Switch-mode operation to have improved efficiency.
2. Sufficient current supplied to load to run the Raspberry Pi 4 with overhead for peripherals.
3. Sufficient charging current to allow better than 1:1 charging/discharging operation.  
4. Charging and sourcing should at least minimally function without uC or RPi intervention.  

The RPi in base configuration does not regularly exceed 1.0A of current draw, and the RPi designers chose a 3A USB power supply to allow overhead for peripherals.  The team wanted to ensure that the Leto project matched that 3.0A power source as a design specification.  This additional current overhead permits the use of attached peripherals such as an SSD, field sensors, overclocking, and active cooling.

The current overhead also permits the battery charge IC to operate well below its thermal limits during regular use.

The charger uC uses internal voltage conversion circuitry to control battery charge curves and its externally connected boost converter circuit to regulate the output voltage.

{{<image src="/rpi/desc_4.png" >}}

Additional selection criteria for the charge controller uC included a boost frequency that would not cause problems with the RPi and onboard ADC for battery state communication to the main uC and RPi.

The battery charge uC communicates with the Leto host uC over I2C.

The connection for a thermistor is provided.  The bq25895 needs an external voltage divider to properly measure the thermistor, and the resistances used on the bq25895 eval board are duplicated here.  

The footprints are provided for an onboard LED indicates the presence of voltage on the output of the charging uC, however, it is not populated.

{{<image src="/rpi/desc_5.png" width="200px" >}}

The charge current can be set manually using the R8 resistor.  The nominal input current max is calculated using I_ILIM = K_ILIM  / R8 = 355 / 220 = 1.61 A.  The ILIM pin is also provided to an ADC capable pin of the uC, as the voltage on ILIM is proportional to the input current, as a fraction of 0.8V.  See the bq25895 datasheet for more info.

{{<image src="/rpi/desc_6.png" >}}

### Battery
The project is designed to use Li-Ion chemistry batteries, with only 1 cell in series.  The nominal battery voltage for Li-Ion chemistry is 3.7-4.2V  The charging IC does not include functionality for cell balancing.  

The Leto project is not designed for use with multicell packs arranged in a series configuration with voltages over 3.7V.

Recognizing the possibility of user error, the battery circuit implemented reverse polarity protection through Q1 and Q6.

{{<image src="/rpi/desc_7.png" >}}

TP1 and TP2 are solder points for wires, which are provided in case the user does not want to use the JST connector that is included.  

The battery thermistor must be connected at J4. The thermistor must be a negative temperature coefficient style.  The recommended unit by TI is a 103AT-2.

{{<image src="/rpi/desc_8.png" width="400px" >}}

### Boost Converter

The boost converter steps the charger output voltage to the target Leto supply voltage.  The TI TPS61088 package provides for a switch current of 10A and internal thermal shutdown protection.

When operated in the target output range of 5V and between 0.5A and 3A, this boost converter achieves 95% efficiency.  The configurable switching frequency permitted the team to design a boost converter that would not cause RPi interference.

NOTE: The enable pin of the TPS61088 cannot be used, as it is not a true-disconnector boost converter.  This means that even when disabled, there is an electrical connection from Vin to Vout through the inductor and an internal FET.  This necessitates using an external load switch. 

{{<image src="/rpi/desc_9.png" >}}

{{<image src="/rpi/desc_10.png" >}}

On board power for the Leto 3.3V control components is provided by a linear regulator, U5:

{{<image src="/rpi/desc_11.png" >}}


### Output Protection

The Leto project provides output protection using a current regulating circuit connected downstream of the boost converter.  The NCP380 regulates output current in a short condition using internal MOFSET switching.

{{<image src="/rpi/desc_12.png" >}}

The OP_Enable signal permits the Leto uC to switch off power to the connected RPi and the FLAG signal indicates a circuit protection state.

The team would like to see additional output signal quantification in future revisions to calculate total power consumption and provide estimated UPS or battery run times.  This feature is not yet implemented due to cost and limits on revision one complexity.


### Output Connections

The Leto RPi power management project is designed to be an under-slung solution. This is to allow the use of a Hat board on the top of the RPi and to not restrict the options for thermal management for the RPi.  The end user is required to assemble provided pogo pins to make connections to the underside of the RPi GPIO header.  Alternatively, the end user can select to solder a standard header, and use the Leto as a RPi Hat. The RPi Hat PCB outline was used specifically to allow this flexibility to the user.

{{<image src="/rpi/desc_13.png" >}}

Note: in the render above all of the GPIO are populated. The end user only needs to solder the required pins identified in the schematic below:

{{<image src="/rpi/desc_14.png" >}}

The Leto board pin connection functionality is as follows:

{{<image src="/rpi/desc_15.png" >}}

### Microcontroller

With the goal of providing a robust option to the user, a mid-range ST microcontroller was selected, the STM32F412.  The ST series was selected for several reasons including a well-developed HAL which would allow easy change to a different MCU if needed, well supported development environment, and ease of implementation to the hardware.  

An ATTiny85 footprint was also included as a backup MCU, in case the ST was not a feasible solution and to provide a second option for users.

### PCB Design

The PCB was designed as a 2 layer board in order to minimize cost.  Power planes were used to reduce the resistance on the high current nodes, and a ground plane was used on the bottom layer.  The smallest passive is an 0402, again to minimize any cost adders.  Additionally anything below 0402 requires a very delicate touch to rework and specialized equipment, so that was avoided.   Also, certain passives were intentionally oversized to 0603 or larger, as they were anticipated to be re-worked by users.

The team discussed the possibility of making hand-solderability a design goal, but because that greatly limits the selection of battery charger and boost converter ICs, that goal was put aside.  

### Use Cases

Generally speaking, any situation that needs a Raspberry Pi to operate on batteries can be a potential use-case for this design.

More specifically, the following use cases were considered:

1. Uninterruptable Power Supply (UPS).  In cases where there is a potential power interruption, the Superpower provides continuous power so the Raspberry Pi does not have an unexpected restart.
2. Portable Pi.  In cases where it is desirable to have a portable Pi or a Pi that will operate for extended periods of time using something other than "commercial power mains", such as amateur radio Field Day competitions.  This can be used to provide a computer that meets this criteria.

### Known Issues with V1 of Leto PCB

1. Wrong footprint used for U5. 

### V2 changes (To be implemented)

1. Add footprint for external oscillator for STM32.
2. Move LDO onto its own sheet.
3. Consider moving I2C connector to I2C2 port. 

### Potential future versions

1. Version to support Solar panels with MPPT.
2. Version to support more than 1S battery configuration.


[github readme](https://github.com/SensorsIot/SuperPower/tree/master/SuperPower-RPi)
