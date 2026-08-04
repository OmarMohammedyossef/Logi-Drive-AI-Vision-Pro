# Breif

This repo introduce an interface with stm32 mcu family using C++. for now we are supporting `stm32f103C8T6` mcu. For specific mcu family support contact us.

##  How to build?

### 1) Installing ARM Compiler

1) Download the compiler from link:
```
https://developer.arm.com/-/media/Files/downloads/gnu-rm/10.3-2021.10/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2?rev=78196d3461ba4c9089a67b5f33edf82a&hash=5631ACEF1F8F237389F14B41566964EC
``` 

**OR** You can use the following command in terminal

```bash
wget -P ~/Downloads https://developer.arm.com/-/media/Files/downloads/gnu-rm/10.3-2021.10/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2?rev=78196d3461ba4c9089a67b5f33edf82a&hash=5631ACEF1F8F237389F14B41566964EC
```

2) Install the compiler by extracting the tar file

```bash

cd ~/Downloads

tar -xvjf gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2

cd /opt

mv ~/Downloads/gcc-arm-none-eabi-10.3-2021.10 .

```

3) Set compiler path to your environment path by adding the following line to ~/.bashrc file

```bash

export PATH="/opt/gcc-arm-none-eabi-10.3-2021.10/bin:$PATH"

source ~/.bashrc

```

4) Check if installing compiler is done successfully

```bash
arm-none-eabi-g++ --help
```


### 2) Build the rep

1) clone the repo:

**http**
```bash
git clone https://github.com/abadr99/stm32f10xxx_cpp_interface.git
```

**ssh**
```
git clone git@github.com:abadr99/stm32f10xxx_cpp_interface.git
```

2) move to it then build it.
```bash
cd ./stm32f10xxx_cpp_interface/dev/
```

```bash
make build
```


---
### 3) Flash Code on the board

1)  Install `ST-Link`

Follow the steps in this link:
```
https://freeelectron.ro/installing-st-link-v2-to-flash-stm32-targets-on-linux/
```

2) Install `openocd`

```bash
 sudo apt install libtool-bin libtool-doc
```

```bash
 sudo apt install openocd
```
 
 
3) After connecting your board to the laptop type this command 
 ```bash
st-info --probe
```
 	
 the o/p must be an info about the STM board (STM32F103)

---
Now you are ready to run the code and debug it from Vs-code
