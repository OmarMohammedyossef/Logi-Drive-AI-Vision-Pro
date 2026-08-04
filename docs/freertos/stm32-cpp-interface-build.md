# Brief

This repo introduces an interface with the STM32 MCU family using C++. For now we are supporting the `stm32f103C8T6` MCU. For specific MCU family support, contact us.

## How to Build?

### 1. Installing the ARM Compiler

1. Download the compiler from this link:

    ```
    https://developer.arm.com/-/media/Files/downloads/gnu-rm/10.3-2021.10/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2?rev=78196d3461ba4c9089a67b5f33edf82a&hash=5631ACEF1F8F237389F14B41566964EC
    ```

    **OR** use the following command in the terminal:

    ```bash
    wget -P ~/Downloads https://developer.arm.com/-/media/Files/downloads/gnu-rm/10.3-2021.10/gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2?rev=78196d3461ba4c9089a67b5f33edf82a&hash=5631ACEF1F8F237389F14B41566964EC
    ```

2. Install the compiler by extracting the tar file:

    ```bash
    cd ~/Downloads
    tar -xvjf gcc-arm-none-eabi-10.3-2021.10-x86_64-linux.tar.bz2
    cd /opt
    mv ~/Downloads/gcc-arm-none-eabi-10.3-2021.10 .
    ```

3. Set the compiler path in your environment by adding the following line to `~/.bashrc`:

    ```bash
    export PATH="/opt/gcc-arm-none-eabi-10.3-2021.10/bin:$PATH"
    source ~/.bashrc
    ```

4. Check that the compiler installed successfully:

    ```bash
    arm-none-eabi-g++ --help
    ```

### 2. Build the Repo

1. Clone the repo:

    **http**

    ```bash
    git clone https://github.com/abadr99/stm32f10xxx_cpp_interface.git
    ```

    **ssh**

    ```
    git clone git@github.com:abadr99/stm32f10xxx_cpp_interface.git
    ```

2. Move into it, then build it:

    ```bash
    cd ./stm32f10xxx_cpp_interface/dev/
    make build
    ```

---

### 3. Flash Code on the Board

1. Install `ST-Link`. Follow the steps in this link:

    ```
    https://freeelectron.ro/installing-st-link-v2-to-flash-stm32-targets-on-linux/
    ```

2. Install `openocd`:

    ```bash
    sudo apt install libtool-bin libtool-doc
    sudo apt install openocd
    ```

3. After connecting your board to the laptop, run:

    ```bash
    st-info --probe
    ```

    The output must show info about the STM board (STM32F103).

---

Now you are ready to run the code and debug it from VS Code.
