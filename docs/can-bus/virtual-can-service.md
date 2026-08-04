
# Service Unit File

create a file :  `/etc/systemd/system/vcan0.service`


```ini
After=network.target

[Service]
Type=simple
Restart=always
# Load the vcan Kernel Module
ExecStartPre=/sbin/modprobe vcan
# reates a virtual CAN device named vcan0.
ExecStartPre=/sbin/ip link add dev vcan0 type vcan
# Brings up the interface so it's ready to send/receive CAN messages.
ExecStart=/sbin/ip link set up vcan0
# Dont Exit From The service
RemainAfterExit=yes


[Install]
WantedBy=default.target

```

## Systemd Command
1. 
```bash
systemctl daemon-reload
```

2.

```bash
systemctl restart vcan0-setup.service
```

OR
```bash
systemctl start vcan0-setup.service
```

3.
```bash
systemctl status vcan0-setup.service
```

