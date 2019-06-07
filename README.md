# STM32F4-Discovery Minimal LED

This project is for developers new to embedded development who want to start developing on MacOS for their STM32F4-Discovery. Simple hello world tutorials are less common in embedded, because of the technical toolchain and environment setup needed, especially if you choose not to use an IDE.

While this C program is specifically for the st32f429i-discovery (http://www.st.com/en/evaluation-tools/32f429idiscovery.html), the guide can still be used for toolchain set-up and flashing of other stm32f4 boards, the code just might not do anything interesting. The program configures two GPIOs as input and output, connected to the board's button and LED respectively. On another board they might be connected to something else, or nothing at all. Read your datasheet to find which port and pin connect to an LED on your board, or attach an external LED to your board's GPIO pins, and adjust toggle.c accordingly.

### What resources do I need and where do I find them??

#### Cross-compiler & Toolchain
You are developing on an 64-bit x86 processor targeting a 32-bit ARM processor and your system C compiler will be targeting x86, not ARM. The solution is to install or compile a cross-compiler that uses your system to compile code for another architecture, in this case ARM. We'll use a GNU Toolchain that includes the cross-compiler along with tools described here: https://launchpadlibrarian.net/287100883/readme.txt.

The GNU ARM Embedded Toolchain is downloaded from https://launchpad.net/gcc-arm-embedded. I downloaded <code>gcc-arm-none-eabi-5_4-2016q3-20160926-mac.tar</code>. Put the file in a location of your choice and extract it:
```shell
tar -jxvf gcc-arm-none-eabi-5_4-2016q3-20160926-mac.tar
```

Once extracted into a directory like <code>gcc-arm-none-eabi-5_4-2016q3</code>, we put this location on our PATH:
```shell
export PATH=$PATH:/path/to/gcc-arm-none-eabi-5_4-2016q3/bin
```
Now we can call <code>arm-none-eabi-gcc</code> to run our cross-compiler, which will be used in the Makefile. Add the above line to your .bashrc to avoid the need to call it for every new session.

#### Libraries
Code interacts with [peripherals](https://en.wikipedia.org/wiki/Peripheral) by reading and writing to memory-mapped registers. The exact method to configure GPIOs in this way is described in the STM32f4's datasheet, but it can be tedious and we'll instead use the ST [Standard Peripheral Library](http://www.st.com/content/ccc/resource/technical/document/user_manual/59/2d/ab/ad/f8/29/49/d6/DM00023896.pdf/files/DM00023896.pdf/jcr:content/translations/en.DM00023896.pdf), which provides a higher-level wrapper.

To start, download the library from http://www.st.com/en/embedded-software/stsw-stm32065.html and put the unzipped <code>STM32F4xx_DSP_StdPeriph_Lib_V1.8.0</code> into the root directory of this repostory. We are expected to define which board we are using for the library. This is done by defining a C preprocessor macro with your board type. For example, open in a text editor <code>STM32F4xx_DSP_StdPeriph_Lib_V1.8.0/Libraries/CMSIS/Device/ST/STM32F4xx/Include/stm32f4xx.h</code>. After the initial comments you'll see a large commented out section that begins 'Uncomment the line below according to the target STM32 device used in your application'. Although intended, instead of modifying the library code, I decided to define this macro outside of <code>stm32f4xx.h</code>, which gives the same result. I do this with the gcc command line option -D, which you'll see in the Makefile.

Please take a moment to read the options in <code>stm32f4xx.h</code> and find the macro that matches your board. For example, I'm using the STM32F429i board and I found the line <code>#define STM32F429_439xx</code>. Then, open the Makefile in your editor and replace where I defined <code>STM32F429_439xx</code> with the appropriate value for your board.


#### STLINK debugger tools
The board comes with an embedded debugging chip called ST-LINK/V2-B. We will utilize this to "flash" our board. That is, to copy our program into the flash storage of the board. To install the tool is easy on a Mac with [Homebrew](https://brew.sh/)(and on other operating systems - https://github.com/texane/stlink has more information for installing on various Linux distributions, BSD, or Windows):

```shell
brew install stlink
```

With the stlink package installed you should have access to the command <code>st-flash</code>. This is used by the Makefile.

### Building and running
To compile: 
```shell
make
```
To flash:
```shell
make flash
```

The result - pressing the blue USER button will toggle LED 13! 

### Credit
1. http://www.wolinlabs.com/blog/linux.stm32.discovery.gcc.html - C and Makefile are adapted from here.
2. https://spin.atomicobject.com/2013/08/10/arm-cortex-m-toolchain/
