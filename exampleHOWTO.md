# HOWTO: Syncthing installation

## Installation

> Installation of Syncthing [1]

### Download the latest version and unpack

* [Link to latest release](https://github.com/syncthing/syncthing/releases/latest)

```bash
    # Create a temporary directory
    mkdir ~/syncthing
    # Change into the directory
    cd ~/syncthing
    # Download the latest binary release using wget and extract using tar
    # This might be a different version - make sure linux-adm64:
    wget https://github.com/syncthing/syncthing/releases/download/v1.22.1/syncthing-linux-amd64-v1.22.1.tar.gz
    tar xvf syncthing-linux-amd64-v1.22.1.tar.gz
```

* Copy syncthing binary to `/usr/bin/`
  * `/usr/bin/` is in the user's PATH
  * `sudo` is required as user cannot normally write to `/usr/bin/`

```bash
    sudo cp syncthing-linux-amd64-v1.22.1/syncthing /usr/bin/
```

### Syncthing systemd user service configuration

> This section describes configuration for systemd for a non-priviledged user (i.e. not root)

* Make sure the directory `~/.config/systemd/user/` exists:
  * Make the directory if it does not

```bash
    mkdir -p ~/.config/systemd/user/
```

* Create the file `~/.config/systemd/user/syncthing.service`:
  * Note - this file is deliberaely not commented as you must comment a unit file in Assignment 1

```ini
[Unit]
Description=Syncthing - Open Source Continuous File Synchronization
Documentation=man:syncthing(1)
StartLimitIntervalSec=60
StartLimitBurst=4

[Service]
ExecStart=/usr/bin/syncthing serve --no-browser --no-restart --logflags=0
Restart=on-failure
RestartSec=1
SuccessExitStatus=3 4
RestartForceExitStatus=3 4

# Hardening
SystemCallArchitectures=native
MemoryDenyWriteExecute=true
NoNewPrivileges=true

# Elevated permissions to sync ownership (disabled by default),
# see https://docs.syncthing.net/advanced/folder-sync-ownership
#AmbientCapabilities=CAP_CHOWN CAP_FOWNER

[Install]
WantedBy=default.target
```

* Enable the service using `systemctl` [2]
  * Note use of `--user` flag; will start on use login

```bash
    systemctl --user enable syncthing.service
```

* Start the service

```bash
    systemctl --user start syncthing.service
```

* Check service is running

```bash
    systemctl --user status syncthing.service
    # Place command output here if necessary
```


### Configure GUI for all addresses

> By default, GUI only listens on `127.0.0.1`.  Change so Azure host can access.

* Edit the file `.config/syncthing/config.xml`

* Change the following section replacing `127.0.0.1` with `0.0.0.0`:
  * This will allow the service to respond to any IP address

```xml
    <gui enabled="true" tls="false" debugging="false">
        <address>127.0.0.1:8384</address>   <!-- Before -->
        <apikey><!-- ... api key ... --></apikey>
        <theme>default</theme>
    </gui>
```

```xml
    <gui enabled="true" tls="false" debugging="false">
        <address>0.0.0.0:8384</address>    <!-- After -->
        <apikey><!-- ... api key ... --></apikey>
        <theme>default</theme>
    </gui>
```

* Restart syncthing:

```bash
    systemctl --user restart syncthing.service
```

* From the Azure host, go to the link using a browser:  [http://192.168.56.111:8384/](http://192.168.56.111:8384/)

* Default sync folder is `~\Sync`

### Cleanup

* Remove downloaded files

```bash
    cd ~
    rm -r ~/syncthing
```

### References

[1] [https://docs.syncthing.net/intro/getting-started.html](https://docs.syncthing.net/intro/getting-started.html)

[2] `systemctl(1)` Unit file control

## END OF DOCUMENT
