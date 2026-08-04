# IR Sensor over CAN

## Code to test the  IR Sensor with can

```cpp
// /**
// * @file main.cpp
// * @author Mohamed Refat
// * @brief
// * @version 0.1
// * @date 2025-02-03
// *
// * @copyright Copyright (c) 2025
// *
// */
// // commit-id:

// #include "utils/Types.h"
// #include "mcal/stm32f103xx.h"
// #include "utils/BitManipulation.h"
// #include "mcal/Pin.h"
// #include "mcal/Gpio.h"
// #include "mcal/Rcc.h"
// #include "mcal/Can.h"
// #include "mcal/Exti.h"
// #include "mcal/Nvic.h"

// using namespace stm32::type;
// using namespace stm32::registers::rcc;
// using namespace stm32::dev::mcal::pin;
// using namespace stm32::dev::mcal::gpio;
// using namespace stm32::dev::mcal::rcc;
// using namespace stm32::dev::mcal::can;
// using namespace stm32::dev::mcal::exti;
// using namespace stm32::dev::mcal::nvic;
// Pin pc13;
// Pin irPin;

// void EXTI1_ISR(void);

// int main() {
//    Rcc::Init();
//    Gpio::Init();
//    Exti::Init();
//    Nvic::Init();

//    // In itialize system clock and external clock source
//     Rcc::InitSysClock();
//     Rcc::SetExternalClock(kHseCrystal);
//     Rcc::Enable(Peripheral::kIOPC);
//     Rcc::Enable(Peripheral::kIOPB);
//     Rcc::Enable(Peripheral::kIOPA);
//     Rcc::Enable(Peripheral::kCAN);

//     CanConfig conf = {
//        .opMode = OperatingMode::kNormal,
//        .testMode = TestMode::kNormal,
//        .priority = FifoPriority::kID,
//        .receivedFifoLock = ReceivedFifo::kUnLocked,
//        .baudRatePrescaler = 1,
//        .sjw = TimeQuanta::kTq2,
//        .bs1 = TimeQuanta::kTq12,
//        .bs2 = TimeQuanta::kTq3,
//        .TTCM = State::kDisable,
//        .ABOM = State::kDisable,
//        .AWUM = State::kDisable,
//        .NART = State::kDisable
//     };

//    Can::Init(conf);

// //    CanTxMsg txMsg = {
// //        .stdId = 0x123,
// //        .extId = 0x00,
// //        .ide   = IdType::kStId,
// //        .rtr   = RemoteTxReqType::kData,
// //        .dlc   = 8,
// //        .data  = {'M', 'O', 'H', 'B', 'M', 'E', 'C', '\0'}
// //    };

// //    CanRxMsg rxMsg;
// //    FilterConfig filterConf = {
// //        .idHigh     = 0x0000,
// //        .idLow      = 0x0000,
// //        .maskIdHigh = 0x0000,
// //        .maskIdLow  = 0x0000,
// //        .fifoAssign = FifoNumber::kFIFO0,
// //        .bank       = 0,
// //        .mode       = FilterMode::kMask,
// //        .scale      = FilterScale::k32bit,
// //        .activation = State::kEnable
// //    };

//     // Set can Tx Rx configuration
//     Pin pa12_can_tx(kPortA, kPin12, PinMode::kAlternativePushPull_10MHz);
//     Pin pa11_can_rx(kPortA, kPin11, PinMode::kInputFloat);
//     Gpio::Set(pa11_can_rx);
//     Gpio::Set(pa12_can_tx);

//     // Set led configuration
//     pc13.SetPinNumber(kPin13);
//     pc13.SetPort(kPortC);
//     pc13.SetPinMode(PinMode::kOutputPushPull_10MHz);
//     Gpio::Set(pc13);
    
//     // set the ir pin configuration
//     irPin.SetPinNumber(kPin1);
//     irPin.SetPort(kPortA);
//     irPin.SetPinMode(PinMode::kInputFloat);
//     Gpio::Set(irPin);

//     // Turn off the Led at pc13 pin
//     Gpio::SetPinValue(pc13, kHigh);


//     // Configure external interrupt on EXTI1, set callback function, and enable interrupt
//     EXTI_Config EXTI1_Config = {kPortA, Line::kExti1, Trigger::kBoth};
//     Exti::SetpCallBackFunction(Line::kExti1, EXTI1_ISR);
//     Exti::Enable(EXTI1_Config); 
//     Nvic::EnableInterrupt(kEXTI1_IRQn);

// //    Can::FilterInit(filterConf);

//    Can::Start();
//    while (1) {
//     //    Can::Transmit(txMsg);
//     //    for(int i=0; i<50000; i++) {
//     //        __asm("NOP");
//     //    }
//     //    Can::Receive(rxMsg, FifoNumber::kFIFO0);
//     //    if (rxMsg.data[2] == 'H') {
//     //        Gpio::SetPinValue(pc13, kHigh);
//     //    } else {
//     //        Gpio::SetPinValue(pc13, kLow);
//     //    }
//    }
// }

// void EXTI1_ISR(void) {
//     if (Gpio::GetPinValue(irPin) == kLow) {
//         Gpio::SetPinValue(pc13, kLow);
			//if ( mode == auto ) {
				//Turn left
			}
			// Send can frame to raspery pi (right ir)
//     }else  {
//         Gpio::SetPinValue(pc13, kHigh);
//     }
// }


```