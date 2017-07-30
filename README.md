# STM32F4-Discovery Minimal LED

This guide is for developers without much embedded experience that don't know where to begin with their new STM32F4-Discovery. Getting simple hello world tutorials running can be a hurdle in embedded, because of the technical toolchain and environment setup needed, especially if you choose not to use an IDE. So I hope to clearly and explicitly describe all of the steps necessary to light up an LED on your STM32F4-Discovery board.

For now, I will treat our program file <code>toggle.c</code> as a black box, but a solid understanding of C will be useful. In the future I will annotate the code and Makefile.

The board I used is: http://www.st.com/en/evaluation-tools/32f429idiscovery.html

### What resources do I need and where do I find them??

#### Cross-compiler & Toolchain
You are developing on an 64-bit x86 processor targeting a 32-bit ARM processor and your system gcc or clang C compilers will be targeting x86, not ARM. The solution is to install or compile a cross-compiler that uses your system to compile code for another system, in this case ARM. We will use a GNU Toolchain that includes the cross-compiler along with some other tools described here: https://launchpadlibrarian.net/287100883/readme.txt.

The GNU ARM Embedded Toolchain can be downloaded from https://launchpad.net/gcc-arm-embedded. The link is on the right. I downloaded <code>gcc-arm-none-eabi-5_4-2016q3-20160926-mac.tar</code>. Put the file in a location of your choice and with the following command extract it:
```shell
tar -jxvf gcc-arm-none-eabi-5_4-2016q3-20160926-mac.tar
```

Once this is extracted into a directory like <code>gcc-arm-none-eabi-5_4-2016q3</code>, we'll want to put the tools it provides on our PATH so that we can call them name.
```shell
export PATH=$PATH:/path/to/gcc-arm-none-eabi-5_4-2016q3/bin
```
That's it. Now we can call <code>arm-none-eabi-gcc</code> to run our cross-compiler, which will be used in the Makefile. Remember to modify your path everytime you open a new shell. Consider adding it to your .bashrc. 

#### Libraries
The ST library we will be using is the [Standard Peripheral Library](http://www.st.com/content/ccc/resource/technical/document/user_manual/59/2d/ab/ad/f8/29/49/d6/DM00023896.pdf/files/DM00023896.pdf/jcr:content/translations/en.DM00023896.pdf). This has what we need to help us program [peripherals](https://en.wikipedia.org/wiki/Peripheral), in this case an LED. 

To get started, download the library from http://www.st.com/en/embedded-software/stsw-stm32065.htmland put the unzipped <code>STM32F4xx_DSP_StdPeriph_Lib_V1.8.0</code> into the root directory of this repostory. To prepare we need take care of one more item of business. We are expected to define which board we are using for the library. This is done by defining a C preprocessor macro with your board type. For example, open in a text editor <code>STM32F4xx_DSP_StdPeriph_Lib_V1.8.0/Libraries/CMSIS/Device/ST/STM32F4xx/Include/stm32f4xx.h</code>. A few dozen lines down you will see a large commented out section that begins 'Uncomment the line below according to the target STM32 device used in your application'. **But** instead of modifying the library code, I decided to define this macro myself before <code>stm32f4xx.h</code> is evaluated by the preprocessor, which results in the same result. I did this by using the gcc command line option -D, for define, which you will see in the Makefile.

Please take a moment to read the options in <code>stm32f4xx.h</code> and find the macro that matches your board. For example, I'm using the STM32F429i board and I found the line <code>#define STM32F429_439xx</code>. Then, open the Makefile in your editor and replace where I defined <code>STM32F429_439xx</code> with the appropriate value for your board.

#### STLINK debugger tools
The board comes with an embedded debugging chip called ST-LINK/V2-B. We will utilize this to "flash" our board. That is, to copy our program into the flash storage of the board. To install the tool is easy on a Mac (and on other operating systems - https://github.com/texane/stlink has more information for installing on various Linux distributions, BSD, or Windows):

```shell
brew install stlink
```

With the stlink package installed you should have access to the command <code>st-flash</code> for flashing your code. This is used by the Makefile.

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

In toggle.c I defined a macro to be the name of another macro. I use this macro to define which pin we are targeting. Try changing the 13 to 14 to target the 14th pin instead, then run <code>make flash</code> again and see what changes:

```C
#define GPIO_Pin_n GPIO_Pin_13
```

### Credit
1. http://www.wolinlabs.com/blog/linux.stm32.discovery.gcc.html - My C and Makefile are adapted directly from this tutorial, but I discarded the custom libraries provided for the purposes of this tutorial.
2. https://spin.atomicobject.com/2013/08/10/arm-cortex-m-toolchain/
