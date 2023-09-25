# Install Node Guide Gear Staging Testnet V7

## [Website](https://www.gear-tech.io/) | [Twitter](https://twitter.com/gear_techs) | [Discord](https://discord.com/invite/7BQznC9uD9) | :satellite:[Explorer](https://telemetry.gear-tech.io/#list/0x92ed36f0a4a26169cba7c6990d51055c76b6b89de268568615a041eebb619a0e)

## Setting up variables

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```

## Save and import variables into system

```bash
echo 'export NODENAME="'${NODENAME}'"' >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update Packages and Depencies

```bash
sudo apt update && sudo apt upgrade -y
```

## Install Depencies

```bash
sudo apt install -y clang build-essential binaryen cmake protobuf-compiler curl
```

## Install Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## Install Wasm Toolchains

```bash
rustup toolchain add nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```

## Download binaries

```bash
wget https://get.gear.rs/gear-nightly-linux-x86_64.tar.xz
tar xvf gear-nightly-linux-x86_64.tar.xz 
rm gear-nightly-linux-x86_64.tar.xz
mv gear /usr/local/bin
```

## Create service

```bash
printf "[Unit]
Description=Gear Node
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/
ExecStart=/usr/local/bin/gear --name \"$NODENAME\" --telemetry-url \"ws://telemetry-backend-shard.gear-tech.io:32001/submit 0\"
Restart=always
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/gear-node.service
```

## Register and start service

```bash
sudo systemctl daemon-reload && \
sudo systemctl enable gear-node && \
sudo systemctl start gear-node && \
sudo systemctl status gear-node
```

## Checking logs

```bash
sudo journalctl -fu gear-node -o cat
```

## Moving the node and Backup

To move the node to a new server you are to backup then restore the following (provided paths are for default Staging Testnet V6 node's parameters):

The network private key of the node:

```bash
$HOME/.local/share/gear/chains/gear_staging_testnet_v7/network/secret_ed25519
```

(optional) The database:

```bash
$HOME/.local/share/gear/chains/gear_staging_testnet_v7/db/full
```

(optional) The service configuration if you've configured the node as a service:

```bash
/etc/systemd/system/gear-node.service
```

If you don't backup the database, you can always synchronize it from scratch, but keep in mind that this process may take some time.

## Update the node with the new version

```bash
wget https://get.gear.rs/gear-nightly-linux-x86_64.tar.xz
sudo tar -xvf gear-nightly-linux-x86_64.tar.xz
rm gear-nightly-linux-x86_64.tar.xz
sudo systemctl stop gear-node
mv gear /usr/local/bin
sudo systemctl restart gear-node
```

## Remove the node

```bash
sudo systemctl stop gear-node
sudo systemctl disable gear-node
sudo rm -rf /root/.local/share/gear
sudo rm /etc/systemd/system/gear-node.service
rm /usr/local/bin/gear
```

## Default ports are

```bash
P2P: 30333
Prometheus: 9615
HTTP RPC: 9933
WebSocket RPC: 9944
```

## Cleaning the storage

Get a list of all sections

```bash
df -h
```

Checking how much space the blockchain database takes up

```bash
du -h $HOME/.local/share/gear/chains/gear_staging_testnet_v6/db/full
```
Clean the database

```bash

sudo systemctl stop gear-node
gear purge-chain
sudo systemctl start gear-node
```

Checking

```bash
df -h
```
