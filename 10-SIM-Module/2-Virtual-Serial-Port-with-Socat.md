	
---
# Virtual-Serial-Port-with-Socat

This guide explains how to create virtual serial ports using `socat`, and how to read and write to them for simulating UART communication between two terminals or applications.

---

## Table-of-Contents

1. [Introduction](#Introduction)
2. [Requirements](#Requirements)
3. [Create-Virtual-Serial-Ports](#Create-Virtual-Serial-Ports)
4. [Read-and-Write-to-Virtual-Serial-Ports](#Read-and-Write-to-Virtual-Serial-Ports)
5. [Bidirectional-Communication-Concept](#Bidirectional-Communication-Concept)
6. [Keep-Socat-Running-in-Background](#Keep-Socat-Running-in-Background)
7. [Optional-Friendly-Names](#Optional-Friendly-Names)
8. [Cleanup](#Cleanup)

---

## Introduction

In embedded systems and serial communication testing, it's often useful to simulate serial ports without physical hardware. `socat` allows you to create two connected pseudo-terminal devices (PTYs) that behave like virtual UART ports.

---

## Requirements

- Linux system
- `socat` installed (install using: `sudo apt install socat`)

---

## Create-Virtual-Serial-Ports

Use the following command to create a pair of connected virtual serial ports:

```bash
socat -d -d PTY,raw,echo=0 PTY,raw,echo=0
```



Example output:

```
2024/05/08 12:34:56 socat[12345] N PTY is /dev/pts/3
2024/05/08 12:34:56 socat[12345] N PTY is /dev/pts/4
```

These two ports (`/dev/pts/3` and `/dev/pts/4`) are now linked: writing to one will be readable from the other.

**Important:** This `socat` command must keep running in the terminal to maintain the connection.

---

## Read-and-Write-to-Virtual-Serial-Ports

You can open each port in a separate terminal and communicate between them:

### Terminal A (Reader):

```bash
cat /dev/pts/4
```

### Terminal B (Writer):

```bash
echo "Hello from PTY 3" | sudo tee /dev/pts/3
```

You should see the message appear in Terminal A.

You can also use tools like:

```bash
screen /dev/pts/3 9600
screen /dev/pts/4 9600
```

---

## Bidirectional-Communication-Concept

The two virtual serial ports are **connected like a cable**:

```
/dev/pts/3 <==> /dev/pts/4
```

- Data written to `/dev/pts/3` appears on `/dev/pts/4`
    
- Data written to `/dev/pts/4` appears on `/dev/pts/3`
    

This allows full-duplex, two-way communication.

---

## Keep-Socat-Running-in-Background

To keep the connection alive after closing the terminal, use:

```bash
nohup socat -d -d PTY,raw,echo=0 PTY,raw,echo=0 > /tmp/socat.log 2>&1 &
```

Check the created ports in `/tmp/socat.log`.

---

## Optional-Friendly-Names

You can create symlinks to make the device names easier to remember:

```bash
ln -s /dev/pts/3 /tmp/ttyV0
ln -s /dev/pts/4 /tmp/ttyV1
```

Now you can use `/tmp/ttyV0` and `/tmp/ttyV1` in your tools or scripts.

---

## Cleanup

To stop the background `socat` process, run:

```bash
pkill socat
```

To remove the friendly names (symlinks):

```bash
rm /tmp/ttyV0 /tmp/ttyV1
```
