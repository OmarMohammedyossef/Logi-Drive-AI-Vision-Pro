# CAN Interrupts

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

//    CanTxMsg txMsg = {
//        .stdId = 0x123,
//        .extId = 0x00,
//        .ide   = IdType::kStId,
//        .rtr   = RemoteTxReqType::kData,
//        .dlc   = 8,
//        .data  = {'M', 'O', 'H', 'B', 'M', 'E', 'C', '\0'}
//    };

//    CanRxMsg rxMsg;
//    FilterConfig filterConf = {
//        .idHigh     = 0x0000,
//        .idLow      = 0x0000,
//        .maskIdHigh = 0x0000,
//        .maskIdLow  = 0x0000,
//        .fifoAssign = FifoNumber::kFIFO0,
//        .bank       = 0,
//        .mode       = FilterMode::kMask,
//        .scale      = FilterScale::k32bit,
//        .activation = State::kEnable
//    };

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
  

//     // Turn off the Led at pc13 pin
//     Gpio::SetPinValue(pc13, kHigh);



//    Can::FilterInit(filterConf);

//    Can::Start();
//    while (1) {
//        Can::Transmit(txMsg);
//        for(int i=0; i<50000; i++) {
//            __asm("NOP");
//        }
//        Can::Receive(rxMsg, FifoNumber::kFIFO0);
//        if (rxMsg.data[2] == 'H') {
//            Gpio::SetPinValue(pc13, kHigh);
//        } else {
//            Gpio::SetPinValue(pc13, kLow);
//        }
//    }
// }






































































/**
 * @file main.cpp
 * @author your name (you@domain.com)
 * @brief 
 * @version 0.1
 * @date 2024-07-08
 * @copyright Copyright (c) 2024
 */
// commit-id: 2dde2078538c395bb8a7b93e0d81ec3afe4602dd
#include "utils/Types.h"
#include "mcal/stm32f103xx.h"
#include "utils/BitManipulation.h"
#include "utils/Util.h"
#include "mcal/Pin.h"
#include "mcal/Gpio.h"
#include "mcal/Rcc.h"
#include "mcal/Can.h"
#include "mcal/Nvic.h"
#include "utils/Logger.h"


using namespace stm32::registers::rcc;
using namespace stm32::dev::mcal::pin;
using namespace stm32::dev::mcal::gpio;
using namespace stm32::dev::mcal::rcc;
using namespace stm32::dev::mcal::can;
using namespace stm32::dev::mcal::nvic;
using namespace stm32::utils::logger;
using namespace stm32::util;
using namespace stm32::type;

CanTxMsg txMsg;
CanRxMsg rxMsg;


Pin pc13;

void TxMailboxCompleteCallback() {
    Gpio::SetPinValue(pc13, kHigh);
}


void RxFifo0Callback() {
    Can::Receive(rxMsg, FifoNumber::kFIFO0);
    Gpio::SetPinValue(pc13, kLow);
    Can::Transmit(txMsg, TxMailboxCompleteCallback);
}


int main() {
    Rcc::Init();
    Gpio::Init();
    Nvic::Init();


    Rcc::InitSysClock();
    Rcc::SetExternalClock(kHseCrystal);
    Rcc::Enable(Peripheral::kCAN);
    Rcc::Enable(Peripheral::kIOPC);
    Rcc::Enable(Peripheral::kIOPA);


    pc13.SetPort(Port::kPortC);
    pc13.SetPinNumber(PinNumber::kPin13);
    pc13.SetPinMode(PinMode::kOutputPushPull_10MHz);
    Gpio::Set(pc13);

    // Set can Tx Rx configuration
    Pin pa12_can_tx(kPortA, kPin12, PinMode::kAlternativePushPull_10MHz);
    Pin pa11_can_rx(kPortA, kPin11, PinMode::kInputFloat);
    Gpio::Set(pa11_can_rx);
    Gpio::Set(pa12_can_tx);


    CanConfig conf = {
       .opMode = OperatingMode::kNormal,
       .testMode = TestMode::kNormal,
       .priority = FifoPriority::kID,
       .receivedFifoLock = ReceivedFifo::kUnLocked,
       .baudRatePrescaler = 1,
       .sjw = TimeQuanta::kTq2,
       .bs1 = TimeQuanta::kTq12,
       .bs2 = TimeQuanta::kTq3,
       .TTCM = State::kDisable,
       .ABOM = State::kDisable,
       .AWUM = State::kDisable,
       .NART = State::kDisable,
       .error = CanError::kNoEr
    };
    Can::Init(conf);


    FilterConfig filterConf = {
       .idHigh     = 0x0000,
       .idLow      = 0x0000,
       .maskIdHigh = 0x0000,
       .maskIdLow  = 0x0000,
       .fifoAssign = FifoNumber::kFIFO0,
       .bank       = 0,
       .mode       = FilterMode::kMask,
       .scale      = FilterScale::k32bit,
       .activation = State::kEnable
    };
    Can::FilterInit(filterConf);

    
    Can::SetCallback(CallbackId::kFifo0MessagePending, RxFifo0Callback);
    Can::EnableInterrupt(Interrupts::kFifo0MessagePending);

    
    Nvic::EnableInterrupt(InterruptID::kUSB_LP_CAN1_RX0_IRQn);
    Nvic::EnableInterrupt(InterruptID::kUSB_HP_CAN1_TX_IRQn);
    
    txMsg.stdId = 0x123;
    txMsg.extId = 0;
    txMsg.ide = IdType::kStId;
    txMsg.rtr = RemoteTxReqType::kData;
    txMsg.dlc = 8;
    for (uint8_t i = 0; i < 8; i++) {
        txMsg.data[i] = i;
    }

    Can::Start();

    Can::Transmit(txMsg, TxMailboxCompleteCallback);
    while (1) {
        
    }
}

```