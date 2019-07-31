AD5940 Firmware Library
======================================================
[AD5940](https://www.analog.com/en/products/ad5940.html)/AD5941 and [ADuCM355](https://www.analog.com/en/products/aducm355.html) is the latest on-chip system for electrochemical and biosensors. It has more than 180 registers and makes application development quite complicated. This library targets to offer a register-less method to operate the chip.

The library is called ad5940lib but both ADuCM355 and AD5940 can use this library because they only have slight differences. Check section *Differences between AD594x/ADuCM355* for details.

There are also example codes based on this library. Check the corresponding repository for details.
# Read Before Use

To make the chip easier to use, some registers are initialized with non-default value. This is done in function @ref AD5940_Initialize()

**Thus it's important to call this function whenever AFE is reset(Power-On-Reset, hardware reset or soft-reset)**

## Default settings in this library
- Disable AFE Watch Dog Timer. You can enable it in user code if there is need.
- Set register 0xC08(not documented) for FIFO read. *This has to be done, otherwise FIFO read command(SPICMD_READFIFO) won't work.*
- Gate unused clocks(eg. test clock) to save power.
- Enable the function to wake up AFE whenever there is bus activity(CS falling edge on AD594x). So we can wakeup AFE by any register read(discard this read).
- Enable the function to retain SRAM data during hibernate
- Enable the function that allows Sequencer to put AFE to hibernate mode. Note, a correct key value must be input to corresponding register. Use API @ref AD5940_SleepKeyCtrlS() to do this.
- Configure register PMBW to zero, so PMBW.bit[3:2] and bit[0] can control system bandwidth and chip power mode.

# Differences between AD594x/ADuCM355
AD5940/AD5941 can be regarded as the AFE(Analog Front End) of ADuCM355. But they have differences.

AD5940 and AD5941 only differs in package, thus has different available pins.
## AD5940 vs AD5941
- AD5940 has 8 AFE GPIOs and AD5941 has only 3.
- All others are exactly same.
## AD594x vs ADuCM355
- *AD594x only has one Low Power channel, while ADuCM355 has two.* You have to select channel0 for AD594x in user code. To choose channel0, configure corresponding struct members with constant LPDAC0 for Low Power DAC selection and LPAMP0 for LP amplifiers selection.
- AGPIOs. AD594x and ADuCM355 use totally different GPIO control register. Considering AD594x use GPIOAx but ADuCM355 use GPIOBx
- ADuCM355 has only two GPIOs connected to AFE and it has PWM function, while not AD594x.
- AD594x's GPIO can be controlled by sequencer,  while not ADuCM355.
- AD594x has two INTC(interrupt controller). ADuCM355 also has two, **AND** INTC0's output is connected to MCU's P2.1 pin inside of the M355 chip. Check file aducm355-examples/common/ADuCM355Port.c to see the code where configure P2.1 as interrupt input.(not ready on the time of writing.)

# How to use it.
There are plenty example codes for both ADuCM355 and AD5940. They use this library, so start from the example code is easier. Check the repositories below for example code.

- [aducm355-examples](https://github.com/analogdevicesinc/aducm355-examples)

- [ad5940-examples](https://github.com/analogdevicesinc/ad5940-examples)

For ADuCM355, define macro CHIPSEL_M355. 

For AD594x, define macro CHIPSEL_594X, and provide basic SPI low level interface. See examples in repository ad5940-examples.

## API naming rules
All APIs are started with 'AD5940_', following with block name that is going to operate.

For functions ended with character 'S'(short of sequencer), it means the registers accessed in this function are all accessible by Sequencer.

```void AD5940_ADCFilterCfgS(ADCFilterCfg_Type *pFiltCfg);```

Above function is for ADC filter configuration and registers modified are sequencer compatible.

## Port to another hardware platform for AD594x
The library only requires a SPI(4 wire) and one interrupt input pin to receive interrupt from GP0 of AD594x.

Below functions should be provided at least to make the library work.

- void      AD5940_CsClr(void);
- void      AD5940_CsSet(void);
- void      AD5940_RstClr(void);
- void      AD5940_RstSet(void);
- void      AD5940_Delay10us(uint32_t time);
- void      AD5940_ReadWriteNBytes(unsigned char *pSendBuffer,unsigned char *pRecvBuff,unsigned long length);

# License
Copyright (c) 2017-2019 Analog Devices, Inc. All Rights Reserved.

This software is proprietary to Analog Devices, Inc. and its licensors.
By using this software you agree to the terms of the associated
Analog Devices Software License Agreement.
