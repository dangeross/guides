[ [Intro](README.md) ] -- [ [Preparations](raspibolt_10_preparations.md) ] -- [ [Raspberry Pi](raspibolt_20_pi.md) ] -- [ [Bitcoin](raspibolt_30_bitcoin.md) ] -- [ [Lightning](raspibolt_40_lnd.md) ] -- [ [Mainnet](raspibolt_50_mainnet.md) ] -- [ [**Bonus**](raspibolt_60_bonus.md) ] -- [ [Troubleshooting](raspibolt_70_troubleshooting.md) ]

------

### Beginner’s Guide to ️⚡Lightning️⚡ on a Raspberry Pi

------

## Bonus guide: Installing LndHub from BlueWallet 
This guide will show you how to install LndHub from BlueWallet https://github.com/BlueWallet/LndHub. 
Wrapper for Lightning Network Daemon. It provides separate accounts with minimum trust for end users. LndHub requires the installation of `redis` and `node` to run.

*Difficulty: medium*

## Install Redis
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
Create the working directory for Redis. We will create it on the external hard disk.
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
$ nano /etc/redis/redis.conf
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
Setup Redis to start automatically on system startup.
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
$ sudo mkdir /usr/local/lib/nodejs/node-v10.15.1
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



