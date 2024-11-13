## STM32 core sources files location

The install directory of Arduino IDE can be found here:

* `/home/\<USERNAME>/.arduino15/` _(Linux)_
* `/Users/\<USERNAME>/Library/Arduino15/` _(Mac)_
* `C:\Users\<USERNAME>\AppData\Local\Arduino15\` _(Windows 10 and 11)_

First make sure you have installed the stm32 core using the
[[Arduino Boards Manager|Getting-Started#About-Boards-manager-concept]].

The stm32 core files can then be found in this directory:

`<Arduino IDE install directory>/packages/STMicroelectronics/hardware/stm32/2.8.1`.

The `2.8.1` changes depending on your installed stm32 core version.

If you use the portable IDE, as described [here](https://www.arduino.cc/en/Guide/PortableIDE),
then the location is here instead:

* `<Arduino IDE install directory>/portable/` _(Linux/Mac)_
* `<Arduino IDE install directory>\portable\` _(Windows 10 and 11)_

## STM32 Tools files location

Go to the Arduino IDE install directory. See above.

Then, go to the `<Arduino install directory>/packages/STMicroelectronics/tools/STM32Tools/2.2.3` directory.

The `2.2.3` changes depending on your installed stm32 tools version.
