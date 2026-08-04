This REAMED contains how to port the `FreeRTOS`with `STM32F103`

## FreeRTOS
**Free RTOS consist of to main Layers**
-  HW Independent Layer
	- Like `list.c` , `task.c`, `queue.c`
-  HW Dependent Layer
	- Like `port.c`

## Getting FreeRTOS Files
- From their [website](https://www.freertos.org/) you can download the latest version of the `FreeRTOS`.
- After Extracting the files we are interested about two folders --> `Source`, `Demo`
- From the Source File we can get all the dependent and independent source and header files

**Here is the tree of the `Source` Directory**

```bash

├── CMakeLists.txt
├── croutine.c
├── event_groups.c
├── queue.c
├── sbom.spdx
├── stream_buffer.c
├── tasks.c
└── timers.c
├── include
│   ├── atomic.h
│   ├── croutine.h
│   ├── deprecated_definitions.h
│   ├── event_groups.h
│   ├── FreeRTOS.h
│   ├── list.h
│   ├── message_buffer.h
│   ├── mpu_prototypes.h
│   ├── mpu_wrappers.h
│   ├── portable.h
│   ├── projdefs.h
│   ├── queue.h
│   ├── semphr.h
│   ├── stack_macros.h
│   ├── StackMacros.h
│   ├── stdint.readme
│   ├── stream_buffer.h
│   ├── task.h
│   └── timers.h
├── list.c
├── manifest.yml
├── portable
│   ├── GCC
│   │   ├── ARM_CM3
│   │   │   ├── port.c
│   │   │   └── portmacro.h
│   ├── MemMang
│   │   ├── heap_1.c
│   │   ├── heap_2.c
│   │   ├── heap_3.c
│   │   ├── heap_4.c
│   │   ├── heap_5.c
│   │   └── ReadMe.url

```

> - As we see we are only interested of these files
> - From `portable` we can get the HW dependent files 
> - From `MemMang` we can get the type of our heap
> - From `include` we can get all the include files


## Modify the MakeFile
After adding all the necessary in the repository, we can add a condition in the makefile to include the `FreeRTOS` files like this:

In `MakeFile`
```makefile
INCLUDE_FREE_RTOS_FILES=1
ifeq ($(INCLUDE_FREE_RTOS_FILES),1)
	C_SOURCES := $(wildcard ./src/freeRTOS/*.c)
	C_SRC_NAMES:= $(notdir $(C_SOURCES))
	SOURCES_COUNT := $(shell echo $$(($(SOURCES_COUNT) + $(words $(C_SOURCES)))))
endif
``` 

in `stm32.mk`
```makefile
CC_FLAGS:= -mthumb -g -Wall -mcpu=$(CPU) -O3 -Werror -mcpu=cortex-m3 -mthumb -ffunction-sections -fdata-sections -fno-exceptions -Wall -Wextra -DDEBUG -DSTM32F103C8Tx -DSTM32F1 --specs=nano.specs --specs=rdimon.specs -DLOGGER

C_OBJS:= $(patsubst %.c,$(OBJDIR)/%.o,$(C_SRC_NAMES))

$(OBJDIR)/%.o : src/freeRTOS/%.c
	@$(shell   mkdir -p $(OBJDIR))
	@$(ARM_CC) $(CC_FLAGS) $(INC) -c $< -o $@
	$(eval SOURCES_CTR=$(shell echo $$(($(SOURCES_CTR)+1))))
	echo "[Makefile][Dev]: [$(SOURCES_CTR)/$(words $(C_SOURCES))] $<"

```


## Simple Example
Here is an example of having 1 task that blinks the led on `pc13`
```main.cpp
/**
 * @file main.cpp
 * @author your name (you@domain.com)
 * @brief 
 * @version 0.1
 * @date 2024-07-08
 * @copyright Copyright (c) 2024
 */
#include "utils/Types.h"
#include "mcal/stm32f103xx.h"
#include "utils/BitManipulation.h"
#include "mcal/Pin.h"
#include "mcal/Gpio.h"
#include "mcal/Rcc.h"
#include "utils/Logger.h"
#include "freeRTOS/FreeRTOS.h"
#include "freeRTOS//task.h"


using namespace stm32::registers::rcc;
using namespace stm32::dev::mcal::pin;
using namespace stm32::dev::mcal::gpio;
using namespace stm32::dev::mcal::rcc;
using namespace stm32::utils::logger;
using namespace stm32::type;

void T1_Handler(void* pvParameters);

int main() {
    TaskHandle_t T1Handle = NULL;
    BaseType_t  xReturned;
    Rcc::Init();
    Gpio::Init();

    Rcc::Enable(Peripheral::kUSART1);
    Rcc::Enable(Peripheral::kIOPC);
    
    Rcc::InitSysClock();
    Rcc::SetExternalClock(kHseCrystal);

    xReturned = xTaskCreate(T1_Handler, "Task1", 100, NULL, 0, &T1Handle);
    if (xReturned == pdPASS) {
        Logger::Info("Task1 created successfully");
    } else {
        Logger::Error("Task1 creation failed");
    }

    /* Start the scheduler. */
    vTaskStartScheduler();
    while (1) {}
}

void T1_Handler(void* pvParameters) {
    (void)pvParameters;  //  Avoids unused variable warning
    Pin pc13(kPortC, kPin13, PinMode::kOutputPushPull_10MHz);
    Gpio::Set(pc13);
    uint8_t toggleFlag = 1;

    for (;;) {
        Logger::Info("Task1 running...");
        if (toggleFlag) {
            Gpio::SetPinValue(pc13, kLow);
            toggleFlag = 0;
        } else {
            Gpio::SetPinValue(pc13, kHigh);
            toggleFlag = 1;
        }
        vTaskDelay(pdMS_TO_TICKS(500));  // Task delay for 500ms
    }
}

```

---
> **BUT TILL NOW THE CODE DOESN'T WORK YET ** 
---
## Problems

Actually There are Two Problems
### 1st: Interrupt service routines(ISRs)
The ISR is defined in the `freeRTOS` code with a name diff from the startup file code
so we have to define them in the `FreeRTOSConfig.h`

```cpp
/* Definitions that map the FreeRTOS port interrupt handlers to their CMSIS
standard names. */
#define vPortSVCHandler    SVC_Handler
#define xPortPendSVHandler PendSV_Handler

/* IMPORTANT: This define is commented when used with STM32Cube firmware, when the timebase source is SysTick,
              to prevent overwriting SysTick_Handler defined within STM32Cube HAL */

#define xPortSysTickHandler SysTick_Handler
```

Also there was a modification in the `FreeRTOSConfig.h`, which is adding the following macros
```cpp
/* Set the following definitions to 1 to include the API function, or zero
to exclude the API function. */
#define INCLUDE_vTaskPrioritySet            1
#define INCLUDE_uxTaskPriorityGet           1
#define INCLUDE_vTaskDelete                 1
#define INCLUDE_vTaskCleanUpResources       0
#define INCLUDE_vTaskSuspend                1
#define INCLUDE_vTaskDelayUntil             0
#define INCLUDE_vTaskDelay                  1
#define INCLUDE_xTaskGetSchedulerState      1
```

### 2nd: wrong  `.bss` initialization
In the startup code one of the tasks of the `Reset_Handler()` is to initialize the `.bss` with zero.

Old code(`stm32f103_startup.cpp):

```cpp

extern uint32_t _sbss;
extern uint32_t _ebss;

void Reset_Handler(void) {
	....
	uint8_t *pDest = reinterpret_cast<uint8_t*>(&_sdata);
	....
    // -- 2] INITIATE.BSS WITH ZEROS
    pDest = reinterpret_cast<uint8_t*>(&_sbss);
	uint32_t bss_size = ebss - sbss; // Size in bytes
    for (uint32_t i = 0; i < bss_size; i++) {
       pDest[i] = 0;
    }
	....
}
```

- **Issue**: The pointer arithmetic `&_ebss - &_sbss` calculates the number of **elements** (not bytes) between `_ebss` and `_sbss`, and the `pDest` is a `uint8_t *`.
    
- **Effect**: If `_sbss` and `_ebss` are pointers to `uint32_t`, the size calculation is incorrect. For example:
    
    - If `.bss` is 8 bytes long, `&_ebss - &_sbss` returns `2` (number of `uint32_t` elements) instead of `8` (number of bytes).
        
    - This leads to incomplete initialization of the `.bss` section, leaving some variables (e.g., `xNextFreeByte` in `heap_1.c`) uninitialized because we are initializing it byte by byte `pDest[i] = 0;`.


The correct code:

```cpp
void Reset_Handler(void) {
	....
    // -- 2] INITIATE.BSS WITH ZEROS
    pDest = reinterpret_cast<uint8_t*>(&_sbss);
    // Cast to uint8_t* for byte-wise operations
    uint8_t* sbss = reinterpret_cast<uint8_t*>(&_sbss);
    uint8_t* ebss = reinterpret_cast<uint8_t*>(&_ebss);
    uint32_t bss_size = ebss - sbss; // Size in bytes
    for (uint32_t i = 0; i < bss_size; i++) {
       pDest[i] = 0;
    }
	....
}
```

- **Fix**:
    
    - Cast `_sbss` and `_ebss` to `uint8_t*` (byte pointers).
        
    - Calculate the size in bytes using `ebss - sbss`.
        
- **Result**:
    
    - The `.bss` section is fully zero-initialized, ensuring all static/global variables (e.g., `xNextFreeByte`) are properly initialized to zero.

### **Why This Works**

1. **Pointer Arithmetic**:
    
    - Subtracting two `uint8_t*` pointers gives the size in bytes.
        
    - This ensures the entire `.bss` section is covered, regardless of the size of the variables it contains.
        
2. **Alignment**:
    
    - Using `uint8_t*` ensures byte-level addressing, which is safe for all architectures.
        
    - If performance is critical, you can switch to word-aligned writes (e.g., `uint32_t*`) after ensuring the `.bss` section is aligned.