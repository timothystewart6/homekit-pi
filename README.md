# homekit-pi
A simple way to get HomeKit working on a Raspberry Pi using `homebridge`

## Getting Started
This is a simple way to get `homebridege` working on a Raspberry Pi. I have tested this on a Raspberry Pi 3 and a Pi Zero. There are some pretty detailed instructions [here](https://github.com/nfarina/homebridge/wiki/Running-HomeBridge-on-a-Raspberry-Pi) however I wanted a simple copy pasta doc to do this in the future.

## Setting up your Pi
Set up your Pi, enable `ssh` and `ssh` into it.

(Obviously you will need to use your own username and IP/DNS here)
```
ssh pi@raspberry.local
```

## Install

### OS

Update Raspbian
```bash
sudo apt-get update
sudo apt-get upgrade
```

Install firmware updater
```bash
sudo apt-get install rpi-update
```

Update firmware
```shell
sudo rpi-update
```

Reboot
```bash
sudo reboot
```

### Dependencies

Install a few OS dependencies
```bash
sudo apt-get install git make g++
```

### Node

Install a precompiled version of `node`.  I am using `8.x` for `ARMv6`.  If you need something different, grab it from [here](https://nodejs.org/en/download/)
```bash
wget https://nodejs.org/dist/v8.11.3/node-v8.11.3-linux-armv6l.tar.xz
tar xJvf node-v8.11.3-linux-armv6l.tar.xz
sudo mkdir -p /opt/node
sudo mv node-v8.11.3-linux-armv6l/* /opt/node/
sudo update-alternatives --install "/usr/bin/node" "node" "/opt/node/bin/node" 1
sudo update-alternatives --install "/usr/bin/npm" "npm" "/opt/node/bin/npm" 1
```

Add global `node` modules to your path

```bash
nano ~/.bashrc
```

Add the path to the end of the file
```bash
PATH=$PATH:/opt/node/bin
```

Reload your `.bashrc`
```bash
source ~/.bashrc
```

Install a few more OS dependencies that `homebridge` will need
```bash
sudo apt-get install libavahi-compat-libdnssd-dev
```


### homebridge

Install `homebridge`
```bash
npm install -g homebridge
```

### Plugins

This is a good time to install plugins.  You can find a good list [here](https://www.npmjs.com/search?q=homebridge-plugin)

Only install [this plugin](https://github.com/rudders/homebridge-platform-wemo) if you use WeMo devices
```bash
npm install -g homebridge-platform-wemo
```

## Configuration

You will now need to configure `homebridge`.  

```bash
nano ~/.homebridge/config.json

```
Here's a sample config.  You should create your own and save it.

`config.json`
```js
{
  "bridge": {
    "name": "Homebridge",
    "username": "CC:22:3D:E3:CE:30", // change this
    "port": 51826,
    "pin": "031-45-154" // change this
  },

  "description": "This is an example configuration file with one fake accessory and one fake platform. You can use this as a template for creating your own configuration file containing devices you actually own.",
  "platforms": [
    {
      "platform": "BelkinWeMo",
      "name": "WeMo Platform"
    }
  ]
}
```

### Test `homebridge`
You should now test `homebridge`.  

You can do so by running
```bash
homebridge
```

This is a good time to set up your devices and test them with the QR code that appears.

### Run at startup

You will want `homebridge` to run at startup.

```bash
sudo nano /etc/init.d/homebridge
```

paste this init script and save it

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides: homebridge
# Required-Start:    $network $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start daemon at boot time
# Description:       Enable service provided by daemon.
### END INIT INFO

dir="/home/pi"
cmd="DEBUG=* /opt/node/bin/homebridge"
user="pi"

name=`basename $0`
pid_file="/var/run/$name.pid"
stdout_log="/var/log/$name.log"
stderr_log="/var/log/$name.err"

get_pid() {
    cat "$pid_file"
}

is_running() {
    [ -f "$pid_file" ] && ps -p `get_pid` > /dev/null 2>&1
}

case "$1" in
    start)
    if is_running; then
        echo "Already started"
    else
        echo "Starting $name"
        cd "$dir"
        if [ -z "$user" ]; then
            sudo $cmd >> "$stdout_log" 2>> "$stderr_log" &
        else
            sudo -u "$user" $cmd >> "$stdout_log" 2>> "$stderr_log" &
        fi
        echo $! > "$pid_file"
        if ! is_running; then
            echo "Unable to start, see $stdout_log and $stderr_log"
            exit 1
        fi
    fi
    ;;
    stop)
    if is_running; then
        echo -n "Stopping $name.."
        kill `get_pid`
        for i in 1 2 3 4 5 6 7 8 9 10
        # for i in `seq 10`
        do
            if ! is_running; then
                break
            fi

            echo -n "."
            sleep 1
        done
        echo

        if is_running; then
            echo "Not stopped; may still be shutting down or shutdown may have failed"
            exit 1
        else
            echo "Stopped"
            if [ -f "$pid_file" ]; then
                rm "$pid_file"
            fi
        fi
    else
        echo "Not running"
    fi
    ;;
    restart)
    $0 stop
    if is_running; then
        echo "Unable to stop, will not attempt to start"
        exit 1
    fi
    $0 start
    ;;
    status)
    if is_running; then
        echo "Running"
    else
        echo "Stopped"
        exit 1
    fi
    ;;
    *)
    echo "Usage: $0 {start|stop|restart|status}"
    exit 1
    ;;
esac

exit 0
```

update permissions
```bash
sudo chmod 755 /etc/init.d/homebridge
sudo update-rc.d homebridge defaults
```

It should now run at startup.  You can test your script by running

```bash
sudo /etc/init.d/homebridge start
```

### Making changes

If you need to register your `homebridge` again

stop `homebridge`

```bash
sudo /etc/init.d/homebridge stop
```

then run it manually
```bash
homebridge
```

then start it again
```bash
sudo /etc/init.d/homebridge start
```

## Thanks
All credit goes to [homebride](https://github.com/nfarina/homebridge) and [homebridge-platform-wemo](https://github.com/rudders/homebridge-platform-wemo).  Thank you!
