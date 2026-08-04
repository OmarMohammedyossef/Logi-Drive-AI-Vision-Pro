# Raspberry Pi Notes

## Connecting the Ethernet to the RPi

To find the IP:

```bash
sudo apt install arp-scan
sudo arp-scan --interface=eno2 --localnet
```

## Arduino

### Wiring Connections

|MCP2515 Module|Arduino|
|---|---|
|VCC|5V|
|GND|GND|
|CS (Chip Select)|10|
|SCK (SPI Clock)|13|
|SI (MOSI)|11|
|SO (MISO)|12|
|INT (Interrupt)|2 (optional)|
