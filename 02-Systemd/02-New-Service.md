Here is a steps to create a new systemd service to my humble server

## 1) Creating service unit file
- The file must be under `/etc/systemd/system/`
- File name must end with `.srevice`



```ini
[Unit]
Description=Qt Server
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=1
ExecStart=/home/mohamedrefat/Desktop/myService/302-NinjaServer
ExecReload=/home/mohamedrefat/Desktop/myService/302-NinjaServer

[Install]
WantedBy=multi-user.target

```

**Explanation of the Unit File:**

- **`[Unit]` section:**
    
    - `Description`: Provides a human-readable description of the service.
    - `After=network.target`: Ensures that the network is up before the service starts.
- **`[Service]` section:**
	- `Type` : Specifies type of the service
    - `ExecStart`: Defines the command to execute.
    - `Restart=on-failure`: Restarts the service if it fails.
    - `RestartSec`: Specifies the delay before restarting.
- **`[Install]` section:**
    
    - `WantedBy=multi-user.target`: Indicates that the service should be started at multi-user system startup(GUI).


---
### 2) Create the service

1. **Place the unit file:** Save the unit file as `/etc/systemd/system/qt-server.service`.
2.  **Reload the daemon:**
    ```bash
    sudo systemctl daemon-reload
    ```
    
2. **Enable the service:**
    ```bash
    sudo systemctl enable qt-server.service
    ```
    
3. **Start the service:**
    ```bash
    sudo systemctl start qt-server.service
    ```



## 3) Test the server

```bash
curl http://localhost:1234
```
**OR**
```bash
curl -X POST -d "message=Alice&age=30" http://localhost:1234/submit_form
```

Now Check the status of the server

```bash
systemctl status qt-server.service
```
