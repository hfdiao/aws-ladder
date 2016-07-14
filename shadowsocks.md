# Dashboard

Edit "Security Group", allow accessing TCP port 8388 from 0.0.0.0/0

# Server
## install pip
```sh
wget https://bootstrap.pypa.io/get-pip.py  --no-check-certificate
sudo python get-pip.py
```

## install shadowsocks
```sh
sudo pip install shadowsocks
```

## config shadowsocks

```js
// file: /etc/shadowsocks/config.json

{
    "server_port":8388,
    "password":"$password",
    "timeout":300,
    "method":"aes-256-cfb",
    "auth": true
}
```

## auto start when system boots

```
sudo useradd shadowsocks
```

```sh
#!/bin/sh

# file: /etc/init.d/shadowsocks

PIDFILE=/var/run/shadowsocks.pid
LOGFILE=/var/log/shadowsocks.log

start(){
        ssserver -c /etc/shadowsocks/config.json -d start --fast-open --workers 3 --pid-file $PIDFILE --log-file $LOGFILE --user shadowsocks
}

stop(){
        ssserver -c /etc/shadowsocks/config.json -d stop
}

case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
reload)
        stop
        start
        ;;
*)
        echo "Usage: $0 {start|reload|stop}"
        exit 1
        ;;
esac
```

```
update-rc.d shadowsocks defaults
```

## start shadowsocks

```
service start shadowsocks
```

# Client
## download

https://shadowsocks.org/en/download/clients.html

## config

```
local port: 1080  
server port: 8388  
encrypt method: aes-256-cfb  
password: $password  
server addr: $aws_public_ip or $aws_public_domain
```

## autoproxy (optional)

https://github.com/hfdiao/confbackup/raw/master/gfw.pac
