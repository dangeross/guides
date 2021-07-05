[ [Intro](README.md) ] -- [ [Preparations](raspibolt_10_preparations.md) ] -- [ [Raspberry Pi](raspibolt_20_pi.md) ] -- [ [Bitcoin](raspibolt_30_bitcoin.md) ] -- [ [Lightning](raspibolt_40_lnd.md) ] -- [ [Mainnet](raspibolt_50_mainnet.md) ] -- [ [**Bonus**](raspibolt_60_bonus.md) ] -- [ [Troubleshooting](raspibolt_70_troubleshooting.md) ]

------

### Beginner’s Guide to ️⚡Lightning️⚡ on a Raspberry Pi

------

## Bonus guide: Installing LndHub from BlueWallet 
This guide will show you how to install LndHub from BlueWallet https://github.com/BlueWallet/LndHub. 
Wrapper for Lightning Network Daemon. It provides separate accounts with minimum trust for end users. LndHub requires the installation of `redis` and `node` to run.

*Difficulty: medium*

## Install Redis
Login as "admin"

Download Redis
```
$ cd /home/admin/download
$ rm -rf /home/admin/download/*
$ wget http://download.redis.io/releases/redis-5.0.3.tar.gz
$ tar xzf redis-5.0.3.tar.gz
$ cd redis-5.0.3/
```
Make & install
```
$ make
$ sudo make install
```
Create the working directory for Redis. We will create it on the external hard disk
```
$ sudo mkdir /mnt/hdd/redis
```
Create the redis user
```
$ sudo adduser --system --group --no-create-home redis
$ sudo chown redis:redis /mnt/hdd/redis
$ sudo chmod 770 /mnt/hdd/redis
```
Make a directory for the `redis.conf` file & copy the default conf file to it.
```
$ sudo mkdir /etc/redis
$ sudo cp redis.conf /etc/redis
```
Edit the `redis.conf` to change the supervised & dir options.
```
$ sudo nano /etc/redis/redis.conf
```
Find `supervised no` and change it to `supervised systemd`
```
# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised systemd
```
Find `dir` and set it to `/mnt/hdd/redis`
```
# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir /mnt/hdd/redis
```

### Autostart Redis
Setup Redis to start automatically on system startup. Edit the system service config for redis
```
$ sudo nano /etc/systemd/system/redis.service
```
```
[Unit]
Description=Redis In-Memory Data Store
After=network.target

[Service]
User=redis
Group=redis
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
ExecStop=/usr/local/bin/redis-cli shutdown
Restart=always

[Install]
WantedBy=multi-user.target
```
Start up redis via the systemctl and test the status
```
$ sudo systemctl start redis
$ sudo systemctl status redis
```
Test redis is running
```
$ redis-cli
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> exit
```
Enable the service
```
$ sudo systemctl enable redis
```
Clean up
```
$ rm -rf /home/admin/download/*
```
## Install Node
Download Node
```
$ cd /home/admin/download
$ wget https://nodejs.org/dist/v10.15.1/node-v10.15.1-linux-armv7l.tar.xz
```
Extract to `usr/local/lib`
```
$ tar -xf node-v10.15.1-linux-armv7l.tar.xz
$ sudo mkdir /usr/local/lib/nodejs/node-v10.15.1
$ mv node-v10.15.1-linux-armv7l.tar.xz/* /usr/local/lib/nodejs/node-v10.15.1
$ sudo tar -xf node-v10.15.1-linux-armv7l.tar.xz -C /usr/local/lib/nodejs/node-v10.15.1
```
Link node version into `/usr/local/bin`
```
$ sudo ln -s /usr/local/lib/nodejs/node-v10.15.1/bin/node /usr/local/bin/node
$ sudo ln -s /usr/local/lib/nodejs/node-v10.15.1/bin/npm /usr/local/bin/npm
```
Test node & npm is linked correctly
```
$ node -v
$ npm -v
```
Clean up
```
$ rm -rf /home/admin/download/*
```

## Port Forwarding and Uncomplicated Firewall
We need to open up port 3000 *on the router* and allow access through `ufw`. Follow the steps as you have done previously opening the port forwarding on your router. Add a new port forwarding configuration with the following settings:

| Application name | External port | Internal port | Internal IP address | Protocol (TCP or UDP) |
| ---------------- | ------------- | ------------- | ------------------- | --------------------- |
| LndHub           | 3000          | 3000          | <YOUR:NODE:IP:ADDR> | TCP                   |

Allowing access in ufw
```
$ sudo ufw allow 3000 comment 'allow LndHub'
$ sudo ufw reload
$ exit
```

## Unlock LND
Make sure your lnd node is unlocked before running LndHub
```
$ lncli unlock
```

## Install LndHub
Switch user to bitcoin
```
$ su - bitcoin
```
Clone the LndHub repository
```
$ git clone https://github.com/BlueWallet/LndHub
```
Copy the `tls.cert` and `admin.macaroon` files into LndHub
```
$ cp .lnd/data/chain/bitcoin/mainnet/admin.macaroon LndHub/
$ cp .lnd/tls.cert LndHub/
```
Edit the LndHub `config.js` file to add bitcoin RPC credentials
```
$ cd LndHub/
$ nano config.js
```
Edit the `config.js` file 
* Change bitcoind.rpc `login:password` to match the rpcuser & rpcpassword in your bitcoin.conf. Your rpcpassword may need to be URL encoded if it contains special characters. You can use an online service like https://meyerweb.com/eric/tools/dencoder/ to encode it (it does not send your rpcpassword anywhere).
* Change all the IP addresses from `1.1.1.1` to `127.0.0.1`
* Change the redis port from `12914` to `6379`
```
let config = {
  bitcoind: {
    rpc: 'http://rpcuser:rpcpassword@127.0.0.1:8332',
  },
  redis: {
    port: 6379,
    host: '127.0.0.1',
    family: 4,
    db: 0,
  },
  lnd: {
    url: '127.0.0.1:10009',
  },
};
```
Install required node modules
```
$ npm config set prefix '~/.npm-global'
$ export PATH=~/.npm-global/bin:$PATH
$ npm install
```
Compile babeljs
```
$ npm install -g @babel/core @babel/cli
$ mkdir build
$ babel ./ --out-dir ./build --copy-files --ignore node_modules
```
Run LndHub
```
$ node build/index.js
```
Check for `'BOOTING UP' 'Listening on port 3000'`

### Autostart LndHub
Setup LndHub to start automatically on system startup. Switch back to the "admin" user or login as "admin"
```
$ exit
```
Edit the system service config for LndHub
```
$ sudo nano /etc/systemd/system/lndhub.service
```
```
[Unit]
Description=LndHub Wrapper for Lightning Daemon
Wants=lnd.service
After=lnd.service

[Service]
WorkingDirectory=/home/bitcoin/LndHub
ExecStart=/usr/local/bin/node build/index.js

User=bitcoin
Group=bitcoin
Type=simple
KillMode=process
LimitNOFILE=128000
TimeoutSec=240
Restart=always
RestartSec=60

[Install]
WantedBy=multi-user.target
```
Start up LndHub via the systemctl
```
$ sudo systemctl enable lndhub
$ sudo systemctl start lndhub
```
Check the lndhub log file for `'BOOTING UP' 'Listening on port 3000'`
```
$ sudo journalctl -f -u lndhub
```

LndHub should now be ready to use in the BlueWallet, but first
* In your brower enter the RaspiBolt IP and port 3000 to see if you can get the default LndHub page, eg: http://192.168.0.10:3000
* In that page you should see the configured LND uri, eg `03fbceea6554714aa4c94648a64d60860d87ac758ca99bdff20e7dce1b0764696d@77.47.67.53:9735`
* That should be your LND node address
* In the BlueWallet app, configure your Lightning Settings to http://LND_IP_ADDRESS:3000
* Add a new Lightning Wallet
* Monitor the `journalctl` to see if the app is making requests





