# -WPA-Supplicant-Guide-and-Tutorial

`wpa_supplicant` is a user-space daemon that allows Linux systems to authenticate and associate with wireless networks using the WPA (Wi-Fi Protected Access) and WPA2 security protocols. It acts as the backend for most Wi-Fi management tools such as `nmcli`, `NetworkManager`, `wpa_gui`, and `connman`.

It is especially useful in headless environments or when fine-grained control over wireless configurations is required.

---

## -Table-of-Contents

1. [What-is-PSK](#What-is-psk)
    
2. [How-wpa_supplicant-Works](#How-wpa_supplicant-works)
    
3. [Differences-Between-nmcli-and-wpa_supplicant](#Differences-between-nmcli-and-wpa_supplicant)
    
4. [Step-1-Scanning-Available-Networks](#Step-1-scanning-available-networks)
    
5. [Step-2-Connecting-to-a-Network](#Step-2-connecting-to-a-network)
    
6. [Step-3-Disconnect-from-a-Network](#Step-3-disconnect-from-a-network)
    
7. [Step-4-Remove-a-Network-from-Config](#Step-4-remove-a-network-from-config)
    
8. [Step-5-Using-a-wpa_supplicant.conf-File](#Step-5-using-a-wpa_supplicant.conf-file)
    
9. [Step-6-Running-wpa_supplicant-in-Daemon-Mode](#Step-6-running-wpa_supplicant-in-daemon-mode)
    
10. [Troubleshooting](#Troubleshooting)
    
11. [References](#References)
    

---

## What-is-PSK

**PSK (Pre-Shared Key)** is the password used to authenticate with WPA/WPA2-secured networks. It is derived from the user-supplied passphrase and the SSID using a key derivation function (PBKDF2).

To generate the raw PSK:

```bash
wpa_passphrase "SSID" "your_wifi_password"
```

This will output a block that can be directly added to a configuration file.

---

## How-wpa_supplicant-Works

`wpa_supplicant` interfaces directly with the wireless drivers and handles all authentication, encryption, and key management. It uses:

- Configuration files (typically `/etc/wpa_supplicant/wpa_supplicant.conf`)
    
- Commands via `wpa_cli`
    
- Interfaces such as D-Bus
    

---

## Differences-Between-nmcli-and-wpa_supplicant

| Feature           | `nmcli`                              | `wpa_supplicant`              |
| ----------------- | ------------------------------------ | ----------------------------- |
| Interface         | High-level CLI (uses wpa_supplicant) | Low-level daemon + CLI        |
| Config Management | Managed by NetworkManager            | Manual config or `wpa_cli`    |
| Simplicity        | Easier                               | More control, more complex    |
| Best use-case     | Desktop, GUI                         | Embedded, headless, scripting |

---

## Step-1-Scanning-Available-Networks

Use `wpa_cli` to initiate a scan:

```bash
sudo wpa_cli scan
sudo wpa_cli scan_results
```

Or use `iwlist`:

```bash
sudo iwlist wlan0 scan | grep ESSID
```

	---

## Step-2-Connecting-to-a-Network

### Method 1: Using `wpa_cli`

```bash
sudo wpa_cli
> add_network
> set_network 0 ssid "YourSSID"
> set_network 0 psk "your_wifi_password"
> enable_network 0
> save_config
> quit
```

### Method 2: Using a Config File

Create `/etc/wpa_supplicant/wpa_supplicant.conf`:

```conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="YourSSID"
    psk="your_wifi_password"
}
```

Connect:

```bash
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
sudo dhclient wlan0
```

---

## Step-3-Disconnect-from-a-Network

```bash
wpa_cli
> disconnect
```

---

## Step-4-Remove-a-Network-from-Config

```bash
wpa_cli
> remove_network 0
> save_config
```

---

## Step-5-Using-a-wpa_supplicant.conf-File

Advanced options can be added to `wpa_supplicant.conf`, such as priority, key management, or EAP for enterprise:

```conf
network={
    ssid="EnterpriseSSID"
    key_mgmt=WPA-EAP
    identity="user"
    password="password"
    eap=PEAP
    phase2="auth=MSCHAPV2"
}
```

---

## Step-6-Running-wpa_supplicant-in-Daemon-Mode

```bash
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
```

To bring down:

```bash
sudo killall wpa_supplicant
```

---

## Troubleshooting

- Ensure your wireless interface is not managed by NetworkManager (if using `wpa_supplicant` directly):
    
    ```bash
    sudo nmcli device set wlan0 managed no
    ```
    
- Check logs:
    
    ```bash
    journalctl -u wpa_supplicant
    ```
    
- Ensure you get an IP:
    
    ```bash
    sudo dhclient wlan0
    ```
    

---

## References

- `man wpa_supplicant`
    
- [https://w1.fi/wpa_supplicant/](https://w1.fi/wpa_supplicant/)
    
- [https://wireless.wiki.kernel.org/en/users/documentation/wpa_supplicant](https://wireless.wiki.kernel.org/en/users/documentation/wpa_supplicant)
    
- [https://wiki.archlinux.org/title/WPA_supplicant](https://wiki.archlinux.org/title/WPA_supplicant)
    

---