# Encoder

## Code to test the Encoder

```cpp

/**
 * @file main.cpp
 * @author Mohamed Refat
 * @brief 
 * @version 0.1
 * @date 2025-05-5
 * 
 * @copyright Copyright (c) 2024
 * 
 */

// commit-id:

#include "mcal/stm32f103xx.h"
#include "utils/Types.h"
#include "utils/BitManipulation.h"
#include "mcal/Pin.h"
#include "mcal/Gpio.h"
#include "mcal/Rcc.h"
#include "mcal/Nvic.h"
#include "mcal/Exti.h"
#include "mcal/Systick.h"
#include "hal/DC_Motor.h"
#include "hal/DC_Motor.h"

using namespace stm32::type;
using namespace stm32::registers::rcc;
using namespace stm32::dev::mcal::pin;
using namespace stm32::dev::mcal::gpio;
using namespace stm32::dev::mcal::rcc;
using namespace stm32::dev::mcal::nvic;
using namespace stm32::dev::mcal::exti;
using namespace stm32::dev::mcal::systick;
using namespace stm32::dev::hal::dc_motor;



// ENCODER_PULSES_PER_REV 
constexpr uint16_t kEncoderCountPerRev = 400;
volatile uint32_t count_per_second = 0;
volatile uint32_t rpm = 0;
volatile uint32_t flag = 0;
	

Pin pc13;

// ISR for EXTI line 0
void EncoderISR();
void CalculateRPM();

int main() {

    Rcc::Init();
    Gpio::Init();
    Nvic::Init();
    Exti::Init();
    Systick::Init();


    // Initialize system clock and external clock source
    Rcc::InitSysClock();
    Rcc::SetExternalClock(kHseCrystal);
    Rcc::Enable(Peripheral::kIOPC);
    Rcc::Enable(Peripheral::kIOPB);
    Rcc::Enable(Peripheral::kIOPA);


    Pin m1_n1(kPortB, kPin8, PinMode::kOutputPushPull_10MHz);
    Pin m1_n2(kPortB, kPin9, PinMode::kOutputPushPull_10MHz);
    Pin m1_ena(kPortA, kPin1, PinMode::kOutputPushPull_10MHz);
    DC_Motor m1(m1_n1, m1_n2);
    Gpio::Set(m1_ena);
    Gpio::SetPinValue(m1_ena, kHigh);
    m1.Stop();

    // Configure GPIO pin A0
    Pin encoderPin(kPortA, kPin0, PinMode::kInputPullDown);
    Gpio::Set(encoderPin);

    pc13.SetPort(kPortC);
    pc13.SetPinNumber(kPin13);
    pc13.SetPinMode(PinMode::kOutputPushPull_10MHz);
    Gpio::Set(pc13);
    Gpio::SetPinValue(pc13, kHigh);

    count_per_second = 0;
    rpm = 0;
    flag = 0;
    // EXTI setup
    EXTI_Config encoderExtiCnf = {kPortA, Line::kExti0, Trigger::kRising};
    Exti::Enable(encoderExtiCnf);
    Exti::SetpCallBackFunction(Line::kExti0, EncoderISR);
    Systick::Delay_By_Exception(1000000, CalculateRPM);
    
    
    m1.ClockWise();
    Nvic::EnableInterrupt(kEXTI0_IRQn);
    Systick::Enable(CLKSource::kAHB_Div_8);
    
    
    while (1) {
        if (flag == 1) {
            // Gpio::SetPinValue(pc13, kLow);
            m1.Stop();
            Systick::InterruptDisable();
            Systick::Disable();
            Nvic::DisableInterrupt(kEXTI0_IRQn);
            rpm = (count_per_second * 60) / kEncoderCountPerRev;
            count_per_second = 0;
            flag = 0;
            Gpio::SetPinValue(pc13, kLow);
            Systick::Delay_By_Exception(1000000, CalculateRPM);
            Systick::Enable(CLKSource::kAHB_Div_8);
            Nvic::EnableInterrupt(kEXTI0_IRQn);
        }
    }
}

// ISR for EXTI line 0
void EncoderISR() {
    count_per_second++;
}

void CalculateRPM() {
    flag = 1;
}
```

