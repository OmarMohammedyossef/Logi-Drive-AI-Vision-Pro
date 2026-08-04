

# 📄 Full Documentation: SIM800 HTTP GET Request via GPRS

---

## 📌 Purpose

This code enables an Arduino to:

- Connect to the internet using a **SIM800 GSM module**
    
- Perform an **HTTP GET** request over **GPRS**
    
- Buffer and print the entire server response
    
- Clean up the connection afterward
    

---

## 🧱 Hardware Setup

|Arduino Pin|SIM800 Pin|
|---|---|
|D7|TX|
|D8|RX|
|GND|GND|
|5V / 3.3V|VCC (based on module specs)|

---

## 🧰 Libraries Used

```cpp
#include <SoftwareSerial.h>
```

- Enables serial communication on pins **7 (RX)** and **8 (TX)**.
    

---

## 🧠 Concepts Covered

|Concept|Explanation|
|---|---|
|GPRS|Internet over mobile network using SIM800|
|APN|Access Point Name, provided by your SIM carrier|
|HTTP GET|Retrieve data from a remote web server|
|AT Commands|Text-based modem command protocol|
|Serial Communication|Arduino ⇆ SIM800 via `SoftwareSerial`|
|Buffering|Assembles a complete response before printing|
|Timeout Handling|Ensures system doesn't freeze waiting for modem|

---

## 📄 Code Structure

---

### 🟩 Global Variables

```cpp
SoftwareSerial sim800(7, 8); // RX, TX
String apn = "etisalat";
String url = "http://www.example.com/";
```

- **`sim800`**: Serial interface to SIM800
    
- **`apn`**: Mobile operator's GPRS gateway name
    
- **`url`**: The target HTTP endpoint (must be HTTP, not HTTPS)
    

---

### 🔧 `setup()`

```cpp
void setup() {
  Serial.begin(9600);
  sim800.begin(9600);
  ...
}
```

#### Step-by-step:

1. Start serial communication with PC and SIM800
    
2. Send AT commands to:
    
    - **Set GPRS bearer parameters**
        
    - **Activate GPRS context**
        
3. Initialize HTTP session
    
4. Set URL and CID (context ID)
    
5. Start HTTP GET (`AT+HTTPACTION=0`)
    
6. Wait for response with `+HTTPACTION`
    
7. Read and print the response if data is available
    
8. Clean up by terminating HTTP and closing GPRS
    

---

### 🔧 `sendCommand(const String &cmd)`

```cpp
void sendCommand(const String &cmd)
```

- Sends an AT command
    
- Waits 1 second
    
- Prints any response received from SIM800
    

Example:

```
>> AT+SAPBR=3,1,"APN","etisalat"
OK
```

---

### 🔧 `waitForHttpAction(...)`

```cpp
bool waitForHttpAction(int &statusCode, int &dataLen)
```

- Waits for the `+HTTPACTION` response from the modem
    
- Parses and extracts:
    
    - `statusCode`: e.g., `200` for OK
        
    - `dataLen`: length of response in bytes
        

Example response:

```
+HTTPACTION: 0,200,291
```

---

### 🔧 `readHttpData(int totalLen)`

```cpp
String readHttpData(int totalLen)
```

- Sends: `AT+HTTPREAD=0,<len>` to request response content
    
- Buffers received characters into a `String`
    
- Waits up to 2 seconds since last character
    

Why this matters:

> The SIM800 sometimes delays response transmission. This method ensures all data is collected before processing.

---

## 🔁 Example Serial Monitor Output

```
Starting HTTP GET via SIM800...
>> AT
OK
...
HTTP Status: 200
Data Length: 291
>> AT+HTTPREAD=0,291
----- Full HTTP GET Response -----
<html>
  <body>Hello from example.com</body>
</html>
----------- End -------------------
```

---

## 📋 AT Command Summary

|Command|Purpose|
|---|---|
|`AT`|Basic test|
|`AT+SAPBR=3,1,"Contype","GPRS"`|Set bearer type to GPRS|
|`AT+SAPBR=3,1,"APN","etisalat"`|Set the APN|
|`AT+SAPBR=1,1`|Activate bearer|
|`AT+SAPBR=2,1`|Show IP (optional)|
|`AT+HTTPINIT`|Start HTTP service|
|`AT+HTTPPARA="CID",1`|Use bearer 1|
|`AT+HTTPPARA="URL","http://..."`|Set the URL|
|`AT+HTTPACTION=0`|Perform GET|
|`+HTTPACTION: 0,<status>,<len>`|GET result|
|`AT+HTTPREAD=0,len`|Read response|
|`AT+HTTPTERM`|Terminate HTTP service|
|`AT+SAPBR=0,1`|Deactivate bearer|

---

## 🔚 Cleanup

Always terminate the HTTP service and bearer to:

- Avoid memory leaks on the SIM800
    
- Free up modem resources
    
- Prepare for the next connection
    

```cpp
sendCommand("AT+HTTPTERM");
sendCommand("AT+SAPBR=0,1");
```

---

## ✅ Final Notes

- Use **HTTP (not HTTPS)** — SIM800 does not support TLS natively
    
- Use **valid APN** for your carrier
    
- Make sure **server is reachable** from mobile network
    
- Responses vary depending on URL and connectivity
    

---

## 🧪 Optional Enhancements

Want more features? I can help you:

- Add retry logic for failed GETs
    
- Store response to SD card
    
- Parse JSON/XML from the response
    
- Use watchdog to recover on lockup
    
- Upgrade to HTTPS with SSL library (SIM800L + external cert)
    

---

## Sending HTTP GET Command

```cpp
#include <SoftwareSerial.h>

SoftwareSerial sim800(7, 8); // RX, TX

String apn = "etisalat";
String url = "http://www.example.com/"; // Must be HTTP, not HTTPS

void setup() {
  Serial.begin(9600);
  sim800.begin(9600);
  delay(1000);

  Serial.println("Starting HTTP GET via SIM800...");

  // 1. Setup GPRS
  sendCommand("AT");
  sendCommand("AT+SAPBR=3,1,\"Contype\",\"GPRS\"");
  sendCommand("AT+SAPBR=3,1,\"APN\",\"" + apn + "\"");
  sendCommand("AT+SAPBR=1,1");
  delay(2000);
  sendCommand("AT+SAPBR=2,1");

  // 2. HTTP GET setup
  sendCommand("AT+HTTPINIT");
  sendCommand("AT+HTTPPARA=\"CID\",1");
  sendCommand("AT+HTTPPARA=\"URL\",\"" + url + "\"");

  // 3. Perform HTTP GET
  sim800.println("AT+HTTPACTION=0");

  int statusCode = -1;
  int dataLen = -1;
  if (waitForHttpAction(statusCode, dataLen)) {
    Serial.print("HTTP Status: ");
    Serial.println(statusCode);
    Serial.print("Data Length: ");
    Serial.println(dataLen);

    if (dataLen > 0) {
      String response = readHttpData(dataLen);
      Serial.println("----- Full HTTP GET Response -----");
      Serial.println(response);
      Serial.println("----------- End -------------------");
    } else {
      Serial.println("⚠️ No data received.");
    }
  } else {
    Serial.println("❌ HTTPACTION timeout or error.");
  }

  // 4. Cleanup
  sendCommand("AT+HTTPTERM");
  sendCommand("AT+SAPBR=0,1");
}

void loop() {
  // Nothing here
}

// Sends an AT command and prints the modem's response
void sendCommand(const String &cmd) {
  sim800.println(cmd);
  Serial.println(">> " + cmd);
  delay(1000);
  while (sim800.available()) {
    Serial.write(sim800.read());
  }
}

// Waits for +HTTPACTION response and extracts status and length
bool waitForHttpAction(int &statusCode, int &dataLen) {
  unsigned long start = millis();
  String line;
  while (millis() - start < 10000) {
    if (sim800.available()) {
      char c = sim800.read();
      if (c == '\n') {
        line.trim();
        if (line.startsWith("+HTTPACTION:")) {
          // Format: +HTTPACTION: 0,<status>,<length>
          int firstComma = line.indexOf(',');
          int secondComma = line.indexOf(',', firstComma + 1);
          if (firstComma > 0 && secondComma > firstComma) {
            statusCode = line.substring(firstComma + 1, secondComma).toInt();
            dataLen = line.substring(secondComma + 1).toInt();
            return true;
          }
        }
        line = "";
      } else {
        line += c;
      }
    }
  }
  return false;
}

// Reads the full HTTP response as a single String
String readHttpData(int totalLen) {
  String response = "";

  sim800.println("AT+HTTPREAD=0," + String(totalLen));
  Serial.println(">> AT+HTTPREAD=0," + String(totalLen));
  delay(100);  // Wait briefly for the modem to respond

  unsigned long lastData = millis();
  while (response.length() < totalLen && millis() - lastData < 2000) {
    if (sim800.available()) {
      char c = sim800.read();
      response += c;
      lastData = millis();  // Reset timeout on data
    }
  }

  return response;
}
```

