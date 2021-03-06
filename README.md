# stm8ef-modbus
[![Travis-CI](https://travis-ci.org/TG9541/stm8ef-modbus.svg)](https://travis-ci.org/TG9541/stm8ef-modbus)

This repository provides a (very) lightweight MODBUS RTU implementation with [STM8 eForth](https://github.com/TG9541/stm8ef/wiki) for "wired" control nodes, e.g. for home automation. The is intended for low-cost STM8S 8bit µCs like the STM8S003F3P6 with 8K Flash and 1K RAM.

Using STM8 Forth for MODBUS has many advantages: while the implementation is extraordinarily compact it gives applications access to many advanced architectural features like independent I/O-locic execution in the background, a CLI (command line interface), compilation of logic the application, and more!

The MODBUS implementation covers basic FCs and implements a subset of [MODBUS V1.1b](http://www.modbus.org/docs/Modbus_Application_Protocol_V1_1b.pdf) which covers use cases of simple MODBUS servers (i.e. dependent end nodes). Right now there is no MODBUS master implementation but the code in this repository can be re-used to write one.

## Supported Boards

The [C0135 board](https://github.com/TG9541/stm8ef/wiki/Board-C0135) is the default target.

![c0135-small](https://user-images.githubusercontent.com/5466977/52519844-fb3c6580-2c61-11e9-8f36-5a031338e6e5.png)

It's easy, however, to build custom targets, e.g. using the $0.80 [MINDEV board](https://github.com/TG9541/stm8ef/wiki/Breakout-Boards#stm8s103f3p6-breakout-board), a cheap relay board, and an RS485 break-out board.

![MINDEV](https://camo.githubusercontent.com/82bd480f176951de9a469e134f543a6570f48597/68747470733a2f2f616530312e616c6963646e2e636f6d2f6b662f485442314e6642615056585858586263587058587136785846585858362f357063732d6c6f742d53544d3853313033463350362d73797374656d2d626f6172642d53544d38532d53544d382d646576656c6f706d656e742d626f6172642d6d696e696d756d2d636f72652d626f6172642e6a70675f323230783232302e6a7067)

## Supported MODBUS Function Codes

`MBSERVER` contains MODBUS function plug-ins with the following function codes (FC):

FC | Description | Support
-|-|-
**1**| **Read Coils** | implemented
**2** | **Read Discrete Inputs** | implemented (limit to "8bit aligned")
**3** | **Read Holding Registers** | implemented (variables in RAM)
**4** | **Read Input Registers** | implemented
**5** | **Write Single Coil** | implemented
6 | Write Single (Holding) Register | (write issue if needed)
**15** | **Write Multiple Coils** | implemented
**16** | **Write Multiple Registers** | implemented

Currently there are no diagnostic functions and communication properties have to be hard coded.

## Installation

The [Getting Started](https://github.com/TG9541/stm8ef/wiki/Breakout-Boards#getting-started) section in the STM8 eForth Wiki provides an introduction to flashing STM8 eForth to a target µC.

Please refer to the [Installation Instructions](https://github.com/TG9541/stm8ef-modbus/wiki/HowTo#installation) in the STM8EF-MODBUS Wiki for build instructions.

## Console

While MODBUS communication uses the STM8S UART, the Forth console communicates through a half-duplex simulated RS232 interface through the `PD1/SWIM` GPIO pin (and a diode). This is made possible by the SWIMCOM STM8 eForth "stock binary" which the makefile pulls from the STM8 eForth Releases. Other CLI communication options, e.g. using simulated full-duplex RxD-TxD lines, require building a custom STM8 eForth binary.

Please refer to the [STM8 eForth Wiki](https://github.com/TG9541/stm8ef/wiki/STM8S-Value-Line-Gadgets#other-target-boards) to learn more about half-duplex CLI communication options and preferred terminal programs.

## Architecture

The software architecture separates hardware abstraction and application in simple layers:

Layer|Source file|Description
-|-|-
5|main.fs|configuration and application layer
4|`MBSERVER`|MODBUS FC plug-ins
3|`MBPROTO`|MODBUS protocol layer
2|`UARTISR`|buffered UART communication
1|`BUSCTRL`|bus access (i.e. RS485 direction control)
0|STM8 eForth|lightweight interactive multi-tasking OS

The code is organized in the following execution domains:
* interrupt service routine for buffered communication
* foreground "idle mode" protocol handler
* fixed-cadence background teask for I/O logic
* foreground CLI provided by the SWIMCOM STM8 eForth

Concerns are nicely separeted - it's even possible to change FC handlers without restarting the application!
