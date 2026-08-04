# HTTP POST Requests

Absolutely, Mohamed! Here's a **complete, in-depth documentation** of your Arduino code that uses a **SIM800 GSM module** to send an **HTTP POST request** over GPRS and read the full response.

---

# 📄 Full Documentation: SIM800 HTTP POST Example

## 📌 Overview

This code:

- Connects a **SIM800 module** to the internet via GPRS using an APN
    
- Sends an **HTTP POST** request to a test server (`http://httpbin.org/post`)
    
- Sends form-encoded data (`key=value`)
    
- Receives and prints the full response from the server
    

---

## 📦 Hardware Connections

|Arduino|SIM800|
|---|---|
|D7|TX|
|D8|RX|
|GND|GND|
|5V/3.3V|VCC (based on your SIM800 board, check manual)|

---

## 🧱 Libraries Used

```cpp
#include <SoftwareSerial.h>
```

- `SoftwareSerial` lets you use pins 7 and 8 as a second serial port to communicate with the SIM800 module, leaving the hardware `Serial` for the PC.
    

---

## 🔧 Code Structure and Explanation

---

### ✅ `SoftwareSerial sim800(7, 8);`

Creates a software serial port:

- **Pin 7** → RX (receives from SIM800 TX)
    
- **Pin 8** → TX (sends to SIM800 RX)
    

---

### ✅ `setup()`

```cpp
void setup() {
  Serial.begin(9600);
  sim800.begin(9600);
  delay(2000);

  sim800PostRequest(apn, postUrl, postData);
}
```

- Starts both serial ports at `9600` baud
    
- Delays for stability
    
- Calls `sim800PostRequest()` with:
    
    - `apn = "etisalat"`
        
    - `postUrl = "http://httpbin.org/post"`
        
    - `postData = "name=mohamed&message=hello"`
        

---

### ✅ `loop()`

```cpp
void loop() {
  // Nothing here
}
```

Unused in this example — all the logic runs in `setup()` once.

---

## 🔁 Main Function: `sim800PostRequest(...)`

```cpp
void sim800PostRequest(const String &apn, const String &postUrl, const String &postData)
```

### ✅ 1. Setup GPRS

```cpp
sendCommand("AT");
sendCommand("AT+SAPBR=3,1,\"Contype\",\"GPRS\"");
sendCommand("AT+SAPBR=3,1,\"APN\",\"" + apn + "\"");
sendCommand("AT+SAPBR=1,1");
sendCommand("AT+SAPBR=2,1");
```

|AT Command|Purpose|
|---|---|
|`AT`|Basic test command. Replies `OK` if working|
|`AT+SAPBR=3,1,"Contype","GPRS"`|Sets bearer profile 1 to GPRS|
|`AT+SAPBR=3,1,"APN","etisalat"`|Sets APN (Access Point Name) for your SIM|
|`AT+SAPBR=1,1`|Opens GPRS bearer|
|`AT+SAPBR=2,1`|Optional: confirms IP address obtained|

---

### ✅ 2. HTTP Setup

```cpp
sendCommand("AT+HTTPINIT");
sendCommand("AT+HTTPPARA=\"CID\",1");
sendCommand("AT+HTTPPARA=\"URL\",\"" + postUrl + "\"");
sendCommand("AT+HTTPPARA=\"CONTENT\",\"application/x-www-form-urlencoded\"");
```

|AT Command|Purpose|
|---|---|
|`AT+HTTPINIT`|Initializes HTTP service|
|`AT+HTTPPARA="CID",1`|Uses bearer profile 1|
|`AT+HTTPPARA="URL",...)`|Sets the request URL|
|`AT+HTTPPARA="CONTENT",...)`|Sets content type of POST data|

---

### ✅ 3. Send POST Body

```cpp
sim800.println("AT+HTTPDATA=" + String(dataLength) + ",10000");
```

|AT Command|Purpose|
|---|---|
|`AT+HTTPDATA=<length>,<timeout>`|Prepares SIM800 to receive `<length>` bytes within `<timeout>` ms|

Then:

```cpp
// Wait for "DOWNLOAD"
while (!sim800.find("DOWNLOAD")) { ... }
sim800.print(postData);
```

- Waits for `"DOWNLOAD"` response
    
- Sends the actual POST body (e.g. `name=mohamed&message=hello`)
    

---

### ✅ 4. Perform POST

```cpp
sim800.println("AT+HTTPACTION=1");
```

|AT Command|Purpose|
|---|---|
|`AT+HTTPACTION=1`|Executes HTTP **POST**|

The SIM800 responds with:

```
+HTTPACTION:1,<status_code>,<data_length>
```

---

### ✅ 5. Get Response Length

```cpp
int responseLength = getHttpResponseLength();
```

Reads the SIM800 output until it sees `+HTTPACTION:1,200,<length>`  
Extracts `<length>` to read the response correctly.

---

### ✅ 6. Read Full Response

```cpp
String response = readHttpResponse(responseLength);
Serial.println(response);
```

Sends:

```cpp
AT+HTTPREAD=0,<length>
```

Then reads that exact number of characters, even if they arrive slowly.

---

### ✅ 7. Cleanup

```cpp
sendCommand("AT+HTTPTERM");
sendCommand("AT+SAPBR=0,1");
```

|AT Command|Purpose|
|---|---|
|`AT+HTTPTERM`|Ends HTTP session|
|`AT+SAPBR=0,1`|Closes GPRS bearer|

---

## 🧰 Utility Functions

### 🔧 `sendCommand(...)`

```cpp
void sendCommand(const String &cmd) {
  sim800.println(cmd);
  Serial.println(">> " + cmd);
  delay(1000);
  while (sim800.available()) {
    Serial.write(sim800.read());
  }
}
```

Sends an AT command and echoes the response to the Serial Monitor.

---

### 🔧 `getHttpResponseLength()`

```cpp
int getHttpResponseLength()
```

- Waits for `+HTTPACTION:` result
    
- Extracts the third number (data length) after two commas
    

Example:

```
+HTTPACTION:1,200,376 → returns 376
```

---

### 🔧 `readHttpResponse(int length)`

```cpp
String readHttpResponse(int length)
```

- Sends: `AT+HTTPREAD=0,<length>`
    
- Waits until all expected characters are received
    
- Returns the response as a string
    

---

## 📤 Output Example (Serial Monitor)

```
>> AT
OK
>> AT+SAPBR=3,1,"Contype","GPRS"
OK
...
>> AT+HTTPACTION=1
OK
+HTTPACTION:1,200,376
>> AT+HTTPREAD=0,376
+HTTPREAD:376
{
  "form": {
    "name": "mohamed",
    "message": "hello"
  },
  ...
}
```

---

## 📌 Concepts Used

|Concept|Explanation|
|---|---|
|GPRS|Mobile data connection via SIM|
|APN|Access Point Name — like a gateway to the internet|
|HTTP POST|Sends data to a server (form body)|
|AT commands|ASCII-based command set used to control modems|
|Serial communication|Arduino ⇄ SIM800 over `SoftwareSerial`|
|Timeout handling|Ensures reliability when waiting for responses|
|Buffering|Stores response in a `String` before printing|

---

## ✅ Summary

This code demonstrates:

- Sending data from Arduino to the internet using only AT commands
    
- Handling variable-length responses
    
- Fully managing a connection session using SIM800
    

---

Let me know if you want to:

- Use HTTPS (TLS)
    
- Send JSON data
    
- Retry on failure
    
- Use hardware serial on Mega or ESP32 for better speed
    

---

## Code

```cpp
#include <SoftwareSerial.h>


SoftwareSerial sim800(7, 8); // RX, TX
// Example POST to httpbin
  String apn = "etisalat";
  String postUrl = "http://httpbin.org/post";
  String postData = "name=mohamed&message=hello";

  
void setup() {
  Serial.begin(9600);
  sim800.begin(9600);
  delay(2000);

  sim800PostRequest(apn, postUrl, postData);
}

void loop() {
  // Nothing here
}


void sim800PostRequest(const String &apn, const String &postUrl, const String &postData) {
  // 1. Setup GPRS bearer profile
  sendCommand("AT");
  sendCommand("AT+SAPBR=3,1,\"Contype\",\"GPRS\"");
  sendCommand("AT+SAPBR=3,1,\"APN\",\"" + apn + "\"");
  sendCommand("AT+SAPBR=1,1");
  delay(2000);
  sendCommand("AT+SAPBR=2,1"); // Optional: show IP

  // 2. Setup HTTP parameters
  sendCommand("AT+HTTPINIT");
  sendCommand("AT+HTTPPARA=\"CID\",1");
  sendCommand("AT+HTTPPARA=\"URL\",\"" + postUrl + "\"");
  sendCommand("AT+HTTPPARA=\"CONTENT\",\"application/x-www-form-urlencoded\"");

  // 3. Prepare POST data
  int dataLength = postData.length();
  sim800.println("AT+HTTPDATA=" + String(dataLength) + ",10000");
  delay(100);

  // Wait for "DOWNLOAD"
  unsigned long start = millis();
  while (!sim800.find("DOWNLOAD")) {
    if (millis() - start > 5000) {
      Serial.println("❌ Timeout waiting for DOWNLOAD");
      return;
    }
  }

  // 4. Send the actual POST body
  sim800.print(postData);
  delay(2000); // allow SIM800 to process

  // 5. Send HTTP POST
  sim800.println("AT+HTTPACTION=1");

  // 6. Read length from +HTTPACTION
  int responseLength = getHttpResponseLength();

  // 7. Read the response
  if (responseLength > 0) {
    String response = readHttpResponse(responseLength);
    Serial.println("----- Full Server Response -----");
    Serial.println(response);
    Serial.println("----------- End ----------------");
  } else {
    Serial.println("❌ Failed to get response length.");
  }

  // 8. Cleanup
  sendCommand("AT+HTTPTERM");
  sendCommand("AT+SAPBR=0,1");
}



String readHttpResponse(int length) {
  String response = "";
  sim800.println("AT+HTTPREAD=0," + String(length));
  delay(100);

  unsigned long lastData = millis();
  while (response.length() < length && millis() - lastData < 2000) {
    if (sim800.available()) {
      char c = sim800.read();
      response += c;
      lastData = millis(); // Reset timeout when data is received
    }
  }

  return response;
}



void sendCommand(const String &cmd) {
  sim800.println(cmd);
  Serial.println(">> " + cmd);
  delay(1000);
  while (sim800.available()) {
    Serial.write(sim800.read());
  }
}

int getHttpResponseLength() {
  String line = "";
  unsigned long start = millis();

  while (millis() - start < 8000) {
    if (sim800.available()) {
      char c = sim800.read();
      if (c == '\n' || c == '\r') {
        line.trim();
        if (line.startsWith("+HTTPACTION:")) {
          int lastComma = line.lastIndexOf(',');
          if (lastComma > 0) {
            String lenStr = line.substring(lastComma + 1);
            return lenStr.toInt();
          }
        }
        line = "";
      } else {
        line += c;
      }
    }
  }
  return 0; // Return 0 on failure
}
```