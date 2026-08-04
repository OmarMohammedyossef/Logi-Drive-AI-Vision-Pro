## Hardware Overview

The SIM900 GSM/GPRS shield is built around the powerful SIM900 chip. This shield has everything you need to connect with an Arduino board, plus some extra cool features that make the most of this special chip.


![image](../images/01-SIM_HW.png)!


![image](../images/02-SIM_HW_Back.png)

---

### The SIM900 shield comes loaded with many useful features. Here are some of the important ones:

- Supports Quad-band: GSM850, EGSM900, DCS1800 and PCS1900
- Connect onto any global GSM network with any 2G SIM
- Make and receive voice calls using an external earphone & electret microphone
- Send and receive SMS messages
- Send and receive GPRS data (TCP/IP, HTTP, etc.)
- Scan and receive FM radio broadcasts
- Transmit Power:
    - Class 4 (2W) for GSM850
    - Class 1 (1W) for DCS1800
- Serial-based AT Command Set
- U.FL and SMA connectors for cell antennae
- Accepts Full-size SIM Card

---

### LED Status Indicators

![image](../images/03-Led-Indecators.png)

**PWR LED**: indicates whether your shield is receiving power.

**Status LED**: indicates the shield’s working status. When this light is on, it means the chip is in working mode and functioning as it should.

**Netlight LED**: blinks in different patterns to tell you about your connection to the cellular network:

- OFF: The SIM900 chip is just starting up (assuming it has power).
- 64ms on, 800ms off: The SIM900 chip is running but hasn’t connected to a cellular network yet.
- 64ms on, 3 seconds off: The SIM900 chip has successfully connected to a cellular network and can send/receive text messages and phone calls.
- 64ms on, 300ms off: Your GPRS data connection is active and working.


---

### Supplying Power

![image](../images/04-SIM900-Power.png)

> **Note:**
> 	**Transmission burst (2A)** means that when SIM900 transmits data, it does so in short, high-power bursts which can require **up to 2 amps of current** for a brief time.


- The SIM900 chip operates within a voltage range of 3.4V to 4.4V. 
- To provide a stable and safe power supply, the shield includes a Micrel MIC29302WU voltage regulator. 


![image](../images/05-SIM900-Jack.png)


The shield has a `5.5mm DC jack` where you can plug in any wall adapter that provides 5V-9V DC power. Next to this jack is a slide switch labeled `EXTERN` that lets you choose your power source. If you want to use an external power supply, simply move the switch as shown.

>**Warning:**
>   Your power supply must be able to provide at least 2 amps of surge current. If your power supply can’t provide enough current, the chip will keep resetting itself and won’t work properly.


---

### UART Communication

- The SIM900 shield communicates with an Arduino using the UART protocol. It supports a baud rate ranging from 1200 bps up to 115200 bps. 

- However, you don’t need to worry about setting the exact speed—The SIM900 has a cool feature called **auto-baud detection** which means the speed at which you send your first `AT command` after resetting will set the speed for all future communication.



![image](../images/06-SIM900-UART_Selection.png)

On the shield, you’ll find UART selection jumpers. These jumpers let you choose which pins the shield uses to receive (RX) and transmit (TX) data. You have two options:

- Software UART: Uses pins D8 and D7
- Hardware UART: Uses pins D1 and D0

![image](../images/07-SIM900-Selection-Jumpers.png)


#### Differences between them:

| Feature              | Hardware UART (D0/D1)       | Software UART (D7/D8)     |     |
| -------------------- | --------------------------- | ------------------------- | --- |
| Speed & reliability  | High (hardware-based)       | Lower (CPU emulated)      |     |
| Serial Monitor usage | Not possible simultaneously | Possible simultaneously   |     |
| Typical usage        | Faster, stable apps         | Debugging and flexibility |     |
|                      |                             |                           |     |


**Recommendation**:  
Use **Software UART** (D7/D8) for development and debugging so you can still use the Serial Monitor. Use **Hardware UART** (D0/D1) for production if you need speed and don't need serial output to the PC.



---

### Speaker and Microphone

The shield comes with two standard 3.5mm jacks—one for earphones and the other for a microphone. With these, you can make and answer phone calls, and even listen to FM radio!

![image](../images/08-SIM900-MIC-Earphone.png)

---
### Antenna

The SIM900 needs an external antenna to work properly. Without it, you won’t be able to make calls, send data, or even run certain AT commands.

![Antenna ](../images/09-SIM-Antenna.PNG)


---
### SIM Socket

On the back of the shield, you’ll find a socket for your SIM card. Any 2G full-size SIM card will work perfectly.

![image](../images/10-SIM900-SIM-Card.png)

---

