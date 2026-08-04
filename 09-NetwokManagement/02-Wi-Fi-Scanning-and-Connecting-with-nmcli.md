
# Wi-Fi-Scanning-and-Connecting-with-`nmcli`

`nmcli` is a command-line interface used to control NetworkManager and manage network connections on Linux systems. It provides a convenient way to scan for Wi-Fi networks, connect to them, and manage profiles. Under the hood, `nmcli` interacts with `wpa_supplicant`, the service responsible for authenticating and managing wireless connections, particularly with WPA/WPA2 secured networks. While `wpa_supplicant` handles the actual connection, `nmcli` acts as a high-level interface for configuring and controlling these connections.

---

## Table-of-Contents

1. [-Prerequisites](#-prerequisites)
2. [Step-1-List-Available-Wi-Fi-Networks](#Step-1-list-available-wi-fi-networks)
3. [Step-2-Connect-to-a-Wi-Fi-Network](#Step-2-connect-to-a-wi-fi-network)
4. [Step-3-Connect-to-Open-(No-Password)-Network](#Step-3-connect-to-open-(no-password)-network)
5. [Step-4-Check-Connection-Status](#Step-4-check-connection-status)
6. [Step-5-Disconnect-from-a-Network](#Step-5-disconnect-from-a-network)
7. [Optional-Forget-a-Saved-Network](#Optional-forget-a-saved-network)
8. [Troubleshooting](#Troubleshooting)
9. [Bonus-Tip-Autoconnect-on-Boot](#Bonus-tip-autoconnect-on-boot)
10. [References](#References)

---

## -Prerequisites

- Linux system with `nmcli` installed (usually pre-installed with NetworkManager)
- A wireless network interface (`wlan0`, `wlp3s0`, etc.)
- Sudo/root access

---

## Step-0 Enable and Disable Wifi

```
nmcli radio wifi on
nmcli radio wifi off
```

---

## Step-1-List-Available-Wi-Fi-Networks

```bash
nmcli dev wifi list
```

**Example Output:**

```
IN-USE  SSID           MODE   CHAN  RATE        SIGNAL  BARS  SECURITY
        HomeNetwork     Infra  6     130 Mbit/s  80      ▂▄▆_  WPA2
        CoffeeShopWiFi  Infra  11    130 Mbit/s  60      ▂▄▆_  WPA2
```

To scan again:
```bash
nmcli dev wifi rescan
```


---

## Step-2-Connect-to-a-Wi-Fi-Network

To connect to a network with a password:

```bash
nmcli dev wifi connect "<SSID>" password "<password>"
```

**Example:**
```bash
nmcli dev wifi connect "HomeNetwork" password "mypassword123"
```

If successful, you’ll see:
```
Device 'wlp3s0' successfully activated with '<UUID>'.
```

Your network profile will be saved and auto-connected in the future.

---

## Step-3-Connect-to-Open-(No-Password)-Network

```bash
nmcli dev wifi connect "<SSID>"
```

---

## Step-4-Check-Connection-Status

```bash
nmcli connection show --active
```

**`nmcli connection show --active`** is _not reliable_ to get just the **current SSID** because:

- `nmcli connection show --active` lists **all active connections**:
    - Ethernet (wired)
    - VPNs
    - Wi-Fi
    - Mobile Broadband
- It doesn’t separate "physical Wi-Fi" from others.
    
- **SSID** is **not even a field** in the output there!  
    (It shows connection names, UUIDs, types... but _not SSIDs_.)
    

**This is better:**
```
nmcli -t -f active,ssid dev wifi | grep '^yes' | cut -d':' -f2
```
- `nmcli dev wifi` focuses **only on Wi-Fi**.
    
- `-f active,ssid` gives whether it's active (`yes`/`no`) and the SSID name.
    
- `grep '^yes'` selects the active one.
    
- `cut` extracts the SSID.

---

## Step-5-Disconnect-from-a-Network

```bash
nmcli connection down id "<SSID>"
```

**Example:**
```bash
nmcli connection down id "HomeNetwork"
```

---

## Optional-Forget-a-Saved-Network

```bash
nmcli connection delete id "<SSID>"
```

---

## Troubleshooting

- Check wireless interface:
  ```bash
  nmcli device status
  ```

- Restart NetworkManager:
  ```bash
  sudo systemctl restart NetworkManager
  ```

- Check if the device is managed:
  ```bash
  nmcli dev show
  ```

---

## Bonus-Tip-Autoconnect-on-Boot

`nmcli` saves the network credentials by default for automatic connection. To make sure:

```bash
nmcli connection modify "<SSID>" connection.autoconnect yes
```

---

## References
- `man nmcli`

---

