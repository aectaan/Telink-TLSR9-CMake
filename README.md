# TLSR9-CMake
CMake example for Telink B91 Generic Starter Kit (TLSR9518A-based)

## Install prerequisites
Update packets.
```
sudo apt update && sudo apt upgrade && sudo apt autoremove
```
Get CMake.
```
sudo apt install -y cmake
```
Get toolchain. 
Telink provides [RISC-V Linux toolchain](http://wiki.telink-semi.cn/tools_and_sdk/Tools/IDE/telink_riscv_linux_toolchain.zip), but there are some libraries missing, so you are able to compile and flash your binary, but unable to run GDB server. Also it's very difficult to download it, because connection to server is very slow and unstable. Instead of it we will get toolchain from the [Andes Technology Github Page](https://github.com/orgs/andestech/repositories?type=all), because TLSR9 uses Andes N25F core.
### Variant 1: get precompiled
##### Mculib version (tested by me)
```
wget https://github.com/andestech/Andes-Development-Kit/releases/download/ast-v3_2_3-release-linux/nds32le-elf-mculib-v5f.txz
mkdir toolchain
tar -xvf nds32le-elf-mculib-v5f.txz -C toolchain
rm nds32le-elf-mculib-v5f.txz
```
##### Newlib version 
```
https://github.com/andestech/Andes-Development-Kit/releases/download/ast-v3_2_3-release-linux/nds32le-elf-newlib-v5f.txz
mkdir toolchain
tar -xvf nds32le-elf-newlib-v5f.txz -C toolchain
rm nds32le-elf-newlib-v5f.txz
```
### Variant 2: build it by yourself
Get toolchain
```
git clone https://github.com/andestech/nds-gnu-toolchain.git
cd nds-gnu-toolchain
git submodule update --init --recursive
sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```
Open `build_elf_toolchain.sh` and replace first 6 rows with following:
```
TARGET=riscv32-elf
PREFIX=`pwd`/nds32le-elf-newlib-v5f
ARCH=rv32imfcxv5
ABI=ilp32f
CPU=n25
BUILD=`pwd`/build-nds32le-elf-newlib-v5f
```
Draw the rest of fckng owl:
```
./build_elf_toolchain.sh
```
### Ncurses
Also you will need ncurses library
```
sudo apt install libncurses5 libncurses5:i386
```
### Get debug and flashing tool
```
wget https://github.com/andestech/Andes-Development-Kit/releases/download/ast-v3_2_3-release-linux/flash.zip
wget https://github.com/andestech/Andes-Development-Kit/releases/download/ast-v3_2_3-release-linux/ice.zip
unzip flash.zip -d toolchain
unzip ice.zip -d toolchain
rm -rf flash.zip ice.zip
cd toolchain/ice/
chmod +x ICEman.sh
./ICEman.sh
```

[Telink RISC-V Linux toolchain](http://wiki.telink-semi.cn/tools_and_sdk/Tools/IDE/telink_riscv_linux_toolchain.zip) also contains ICEman and flashing tool.

### Get example SDK
Instructions from Telink are available [here](http://wiki.telink-semi.cn/wiki/IDE-and-Tools/Compiler_Linux/).
```
wget http://wiki.telink-semi.cn/tools_and_sdk/Driver/B91_Driver_SDK.zip
mkdir driver_sdk
unzip B91_Driver_SDK.zip -d driver_sdk/
rm  B91_Driver_SDK.zip
```
We will build UART demo as an example.
### Build
```
cd app && mkdir build && cd build
cmake ..
make
```
You also may use `Ctrl+Shift+B` hotkey.
### Debug
In the `launch.json` you will find two configurations, for [Native Debug VSCode extension](https://marketplace.visualstudio.com/items?itemName=webfreak.debug) and [Cortex-Debug VSCode extension](https://marketplace.visualstudio.com/items?itemName=marus25.cortex-debug). Both of them are tested and works well with TLSR9518A.