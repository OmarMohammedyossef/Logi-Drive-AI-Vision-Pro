	 ## connecting the ethernet to the rpi
to find the ip:
`sudo apt install arp-scan`
then 
```
sudo arp-scan --interface=eno2 --localnet
```


## Arduno
### **Wiring Connections:**

|MCP2515 Module|Arduino|
|---|---|
|VCC|5V|
|GND|GND|
|CS (Chip Select)|10|
|SCK (SPI Clock)|13|
|SI (MOSI)|11|
|SO (MISO)|12|
|INT (Interrupt)|2 (optional)|
