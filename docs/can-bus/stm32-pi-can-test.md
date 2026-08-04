# STM32 <-> Raspberry Pi CAN Test

```
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
#include "utils/JSON.h"
#include "utils/String.h"


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
volatile uint32_t json_count = 0;
String json_str;
String rx_json_str;
Pin pc13;
volatile int number_of_frames = 0;
void TxMailboxCompleteCallback() {
    bool chunk_has_data = false;
    for (uint8_t i = 0; i < 8; i++) {
        if (json_count < json_str.size()) {
            txMsg.data[i] = json_str[json_count++];
            chunk_has_data = true;
        } else {
            txMsg.data[i] = '\0';
        }
    }
    if (chunk_has_data) {
        Can::Transmit(txMsg, TxMailboxCompleteCallback);
    } else {
        // end of chunks
        Gpio::SetPinValue(pc13, kLow);
        json_count = 0;
    } 
}


void handle_received_pair(String &key, String &value) {
    if (key[0] == 'M' || value[0]) {
        Gpio::SetPinValue(pc13, kHigh);
    }
}

void RxFifo0Callback() {
    number_of_frames++;
    Can::Receive(rxMsg, FifoNumber::kFIFO0);
    static bool opening_parenthes = false;
    if (rxMsg.data[0] == '{') {
        opening_parenthes = true;
    }
    if (opening_parenthes) {
        int i = 0;
        while ((i < 8) && (rxMsg.data[i] != '}')) {
            String str(rxMsg.data[i]);
            rx_json_str += str; 
            i++;
        }
        if ((i < 8) && rxMsg.data[i] == '}' && opening_parenthes) {
            opening_parenthes = false;
            rx_json_str += String("}");
            JsonUtil json;
            json.parse(rx_json_str, handle_received_pair);
        }
    }
    Gpio::SetPinValue(pc13, kLow);
}


// void RxFifo0Callback() {
//     Can::Receive(rxMsg, FifoNumber::kFIFO0);
//     static bool opening_parenthes = false;
//     if (rxMsg.data[0] == static_cast<uint8_t>('{')) {
//         opening_parenthes = true;
//     }
//     if (opening_parenthes) {
//         int i = 0;
//         while (rxMsg.data[i] != static_cast<uint8_t>('}') && i < 8) {
//             String str(reinterpret_cast<const char*>(rxMsg.data[i]),1);
//             rx_json_str += str; 
//             i++;
//         }
//         if (rxMsg.data[i] == static_cast<uint8_t>('}') && opening_parenthes) {
//             opening_parenthes = false;
//             rx_json_str += String("}");
//             JsonUtil json;
//             json.parse(rx_json_str, handle_received_pair);
//         }
//     }
//     Gpio::SetPinValue(pc13, kLow);
// }



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

    Pin pa12_tx(kPortA, kPin12, PinMode::kAlternativePushPull_2MHz);
    Pin pa11_rx(kPortA, kPin11, PinMode::kInputFloat);
    Gpio::Set(pa12_tx);
    Gpio::Set(pa11_rx);

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

    JsonUtil json;
    json.add(String("P0101"), String("Can bus communication problem"));
    json.add(String("P0300"), String("Engine misfire detected"));
    json.add(String("C1234"), String("IR1 sensor malfunction (LKA system)"));
    json_str = json.build();
    
    //-- First chunk of data
    json_count = 0;

    Gpio::SetPinValue(pc13, kHigh);
    Can::Start();
    // First chunk
    TxMailboxCompleteCallback();
    while (1) {

    }
}
  

/* 

cansend can0 123#7b22503031303122
cansend can0 123#3a202243616e2062
cansend can0 123#757320636f6d6d75
cansend can0 123#6e69636174696f6e
cansend can0 123#2070726f626c656d
cansend can0 123#222c202250303330
cansend can0 123#30223a2022456e67
cansend can0 123#696e65206d697366
cansend can0 123#6972652064657465
cansend can0 123#63746564222c2022
cansend can0 123#4331323334223a20
cansend can0 123#224952312073656e
cansend can0 123#736f72206d616c66
cansend can0 123#756e6374696f6e20
cansend can0 123#284c4b4120737973
cansend can0 123#74656d29227d0000

*/


/* 

cansend can0 123#7B2022313233223A
cansend can0 123#20224D6F68616D65
cansend can0 123#64222C202243414E
cansend can0 123#313334222C202245
cansend can0 123#72726F72227D0000

*/
```