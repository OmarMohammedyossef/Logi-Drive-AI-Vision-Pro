# Virtual CAN Bus Setup

To create a **virtual CAN bus** on **Ubuntu**, you can use **SocketCAN**, a Linux kernel module that allows you to use CAN devices as network interfaces. Here's a step-by-step guide to create a virtual CAN bus and interact with it.

---

## **Steps to Create a Virtual CAN Bus on Ubuntu**

### **1. Install Required Packages**

Run the following commands to install the required tools:

```bash
sudo apt-get update
sudo apt-get install can-utils
sudo apt install libsocketcan-dev
```

> **Explanation:**  
> `can-utils` provides useful command-line tools for CAN bus, such as `candump`, `cansend`, and `canplayer`.

---

### **2. Load the vcan Kernel Module**

The **vcan** (virtual CAN) module is required to create a virtual CAN interface.

```bash
sudo modprobe vcan
```

> **Explanation:**  
> This command loads the `vcan` kernel module, which acts as a virtual CAN device.

---

### **3. Create a Virtual CAN Interface**

Use the following commands to create and activate a **vcan0** interface.

```bash
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

> **Explanation:**
> 
> - `ip link add dev vcan0 type vcan`: Creates a virtual CAN device named **vcan0**.
> - `ip link set up vcan0`: Brings up the interface so it's ready to send/receive CAN messages.

---

### **4. Verify the CAN Interface**

Check if **vcan0** is active and running:

```bash
ip link show vcan0
```

You should see output similar to:

```
4: vcan0: <NOARP,UP,LOWER_UP> mtu 72 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/can
```

> **Explanation:**  
> This confirms that **vcan0** is active. Look for `UP` and `LOWER_UP` in the output.
>  The output of the `ip addr` command shows the state as `UNKNOWN`, instead of `UP`. That’s normal for a virtual CAN interface on Linux.
---

### **5. Send and Receive CAN Messages**

To send and receive messages on the **vcan0** interface, you can use the following **can-utils** tools.

#### **5.1. Open a Terminal to Listen for CAN Messages**

```bash
candump vcan0
```

> **Explanation:**  
> This listens for incoming CAN messages on **vcan0** and prints them.

---

#### **5.2. Send a Test CAN Message**

Open another terminal and send a message:

```bash
cansend vcan0 123#1122334455667788
```

> **Explanation:**
> 
> - `cansend`: Sends a message on the CAN bus.
> - `vcan0`: The CAN interface.
> - `123#1122334455667788`: A CAN frame with ID `123` and data bytes `11 22 33 44 55 66 77 88`.

If everything works, you should see the message appear in the terminal where `candump vcan0` is running.

---

### **6. Automate Virtual CAN Setup (Optional)**

If you'd like to make the **vcan0** interface available automatically after a system reboot, you can add the following lines to a file (like `/etc/rc.local` or a systemd script):

```bash
#!/bin/bash
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0
```

Make it executable:

```bash
sudo chmod +x /etc/rc.local
```

