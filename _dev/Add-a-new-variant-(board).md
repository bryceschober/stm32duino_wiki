[[/img/Tips-icon.png|alt="Tips"]]Example of all below steps are shown in this PR: [Add Nucleo-F207ZG](https://github.com/stm32duino/Arduino_Core_STM32/pull/63) (ignore the 2 last commits)

# Create a new variant
Go to the '_**variant**_' folder of the STM32 core.<br>
Follow this page: [Where are sources](https://github.com/stm32duino/wiki/wiki/Where-are-sources#stm32-core-sources-files-location)

## 1 - Create a copy of the _**stm32/variants/board_template**_ folder with a name of your choice.

**Example**: To add variant for the [Nucleo-F207ZG](http://www.st.com/en/evaluation-tools/nucleo-f207zg.html)

`cp -a board_template NUCLEO_F207ZG` (_linux_)

[[/img/Tips-icon.png|alt="Tips"]]It's also possible to copy one of the most similar board variant<br>

## 2 - Generate the pin mapping

[[/img/Warning-icon.png|alt="Warning"]] Prerequisites:
* [Python](https://www.python.org/) is required to use the script!
* [STM32CubeMX ](http://www.st.com/en/development-tools/stm32cubemx.html) is required as script parses MCU xml file description provided with the tool.

Go to the '_**src/genpinmap/**_' folder of the STM32 Tools.<br>
Follow this page: [Where are sources](https://github.com/stm32duino/wiki/wiki/Where-are-sources#stm32-tools-files-location)
or get it from [Arduino_Tools github repo](https://github.com/stm32duino/Arduino_Tools/tree/master/src/genpinmap)

**_genpinmap_arduino.py_** script allows to generate the pins mapping of an STM32 MCU.

> USAGE: genpinmap_arduino.py _\<BOARD_NAME\> \<product xml file name\>_<br>
>        _\<BOARD_NAME\>_ is the name of the board as it will be named in variant folder<br>
>        _\<product xml file name\>_ is the STM32 file description in [STM32CubeMX](http://www.st.com/en/development-tools/stm32cubemx.html)

[[/img/Warning-icon.png|alt="Warning"]]This xml file contains non alpha characters in its name, you should call it with quotes!<br>
[[/img/Warning-icon.png|alt="Warning"]] Script uses default  [STM32CubeMX](http://www.st.com/en/development-tools/stm32cubemx.html) installation directory. If you changed it, update the installation path in the script.<br>
[[/img/Tips-icon.png|alt="Tips"]] _\<product xml file name\>_ could be find in **_\<STM32CubeMX\>/db/mcu_** folder<br>

**Example** for the [Nucleo-F207ZG](http://www.st.com/en/evaluation-tools/nucleo-f207zg.html):

`python genpinmap_arduino.py NUCLEO_F207ZG "STM32F207Z(C-E-F-G)Tx.xml"`

Copy the generated **_src/genpinmap/Arduino/\<BOARD_NAME\>/PeripheralPins.c_** file in the variant folder created.

**Example** for the [Nucleo-F207ZG](http://www.st.com/en/evaluation-tools/nucleo-f207zg.html):

copy<br>
**_src/genpinmap/Arduino/NUCLEO_F207ZG/PeripheralPins.c_**<br>
to<br>
**_variant/NUCLEO_F207ZG/_**

## 3 - Review pins mapping
 
After **_PeripheralPins.c_** generation, review it carefully.<br>
Comment a line if the pin is generated several times for the same IP or<br>
if the pin should not be used (overlaid with some HW on the board, for instance)

Use the related user manual to define the best pins mapping:<br>
**Example** for the [Nucleo-F207ZG](http://www.st.com/en/evaluation-tools/nucleo-f207zg.html):<br>
[UM1974: STM32 Nucleo-144 boards](http://www.st.com/resource/en/user_manual/dm00244518.pdf)<br>
    
[[/img/Tips-icon.png|alt="Tips"]] It is also possible to check the equivalent file for mbed os:
[ST-Nucleo-F207ZG](https://developer.mbed.org/platforms/ST-Nucleo-F207ZG/)

## 4 - Review pins and signals pins numbering
Edit the **_variant.h_** and **_variant.cpp_** file.<br>
In **_variant.cpp_**:<br>
1. Fill the array `const PinName digitalPin[]`. This array allows to wrap Arduino pin number(Dx or x)
to STM32 PinName (PYx).
2. Declare Serial instance and define as many Serial instance as desired

In **_variant.h_**:<br>
1. Align the number of PinName defined above in `digitalPin[]` array in **_variant.h_**.<br>
This is a one to one mapping, i.e. the first enum value is the first element in the array.
2. Review macros to point to the right pin number: `LED_BUILTIN, MOSI, MISO, SCLK, SDA, SCL,...`<br>
[[/img/Tips-icon.png|alt="Tips"]] Here, you can add custom macro to fit your needs<br>

## 5 - System Clock configuration
In **_variant.cpp_**, `void SystemClock_Config(void)` need to be defined.<br>
It could be generated thanks [STM32CubeMX ](http://www.st.com/en/development-tools/stm32cubemx.html) or <br>
copied from a [STM32CubeYY](http://www.st.com/en/embedded-software/stm32cube-embedded-software.html?querycriteria=productId=LN1897) project examples 
(where 'YY' could be F0, F1, F2, F3, F4, F7, L0, L1, L4)

From [STM32CubeMX ](http://www.st.com/en/development-tools/stm32cubemx.html):
1. Run [STM32CubeMX ](http://www.st.com/en/development-tools/stm32cubemx.html), create a _New Project_ and select the targeted MCU or the board if listed.
2. Go to **_Pinout_** tab, enable desired peripherals: `I2C, SDIO, SPI, USB,...`
3. Configure the clock:
    * If the board has an external crystal: set in **_Pinout_** tab `RCC->HSE` peripheral to `Crystal`.<br>
In **_Clock Configuration_**, set `HSE Input frequency` to the crystal one then set `PLL Source Mux` to `HSE`.<br>
    * Set `HCLK` to the maximum frequency, [STM32CubeMX ](http://www.st.com/en/development-tools/stm32cubemx.html) will automatically configure the clock tree and resolve conlict if any.
4. Generate code. Set **_Toolchain/IDE_**: `SW4STM32`.<br>
[[/img/Tips-icon.png|alt="Tips"]] Clock configuration for peripherals will be also generated
5. Copy the `void SystemClock_Config(void)` generated in `src/main.c` to **_variant.cpp_**

## 6 - Update ldscript.ld
It could be generated thanks [STM32CubeMX ](http://www.st.com/en/development-tools/stm32cubemx.html) or <br>
copied from a [STM32CubeYY](http://www.st.com/en/embedded-software/stm32cube-embedded-software.html?querycriteria=productId=LN1897) project examples 
(where 'YY' could be F0, F1, F2, F3, F4, F7, L0, L1, L4)<br>
Replace the **_ldscript.ld_** in variant folder by the `STM32YYxxxxxx_FLASH.ld` generated by [STM32CubeMX ](http://www.st.com/en/development-tools/stm32cubemx.html) in the root folder.<br>
**Example** for the [Nucleo-F207ZG](http://www.st.com/en/evaluation-tools/nucleo-f207zg.html): `STM32F207ZGTx_FLASH.ld`

## 7 - HAL configuration
It could be generated thanks [STM32CubeMX ](http://www.st.com/en/development-tools/stm32cubemx.html) or <br>
copied from a [STM32CubeYY](http://www.st.com/en/embedded-software/stm32cube-embedded-software.html?querycriteria=productId=LN1897) project examples 
(where 'YY' could be F0, F1, F2, F3, F4, F7, L0, L1, L4)<br>
Copy the `stm32yyxx_hal_conf.h` file generated in `Inc/` folder to variant folder. (where `yy` is the MCU family)<br>
**_stm32yyxx_hal_conf.h_** in variant folder can be deleted.<br>
**Example** for the [Nucleo-F207ZG](http://www.st.com/en/evaluation-tools/nucleo-f207zg.html): `stm32f2xx_hal_conf.h`

Then edit copied file in order to:
 * Remove line: `#include "main.h"`
 * Enable/Disable (un)desired HAL modules by (un)commenting line like:
 `#define HAL_XXX_MODULE_ENABLED`
 where "_XXX_" is the feature (`ETH, I2S,...`)
 * Adjust `HSE/HSI` values adaptation if needed
 * Update any other configurations

## 8 - ~~Declare the startup file (TODO: generate full list of possible startup file to avoid this step)~~
~~Edit **_cores/arduino/stm32/stm32_def_build.h_** and add the `CMSIS_STARTUP_FILE` definition.<br>
**Example** for the [Nucleo-F207ZG](http://www.st.com/en/evaluation-tools/nucleo-f207zg.html):<br>
`#elif defined(STM32F207xx)`<br>
`#define CMSIS_STARTUP_FILE "startup_stm32f207xx.s"`~~<br>
This step is no more required as all startup files are listed. See [Arduino_Core_STM32#70](https://github.com/stm32duino/Arduino_Core_STM32/issues/70).

## 9 - Declare the variant
It still to add the menu and add relevant informations (Flash and SRAM sizes, cpu freq,...)<br>
[[/img/Tips-icon.png|alt="Tips"]] See: [Arduino Boards.txt specifications](https://github.com/arduino/Arduino/wiki/Arduino-IDE-1.5-3rd-party-Hardware-specification#boardstxt) for further options.<br>
Edit **_boards.txt_** file, then:<br>
1. Copy one section which is the most similar to your board
2. Rename all `menu.board_part_num.<old_board_name>` by `menu.board_part_num.<new_board_name>`
3. Update `build.mcu=` and `build.cmsis_lib_gcc=` to the correct cortex-mX version
4. Update `build.series=` to the correct `STM32YYxx` (where `YY` is the MCU serie)
5. Update `build.product_line=` to the correct `STM32YYXXxx` MCU version.
6. Update `upload.maximum_size=` and `upload.maximum_data_size=` to the correct Flash and SRAM sizes
7. Update `build.f_cpu=` to the right system Core clock

## 10 - Restart
Restart Arduino IDE and try your new board with [[Blink-example]]
