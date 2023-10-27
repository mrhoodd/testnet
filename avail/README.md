# Install Guide Avail

## [Website](https://www.availproject.org/) | [Discord](https://discord.gg/y6fHnxZQX8) | [Twitter](https://twitter.com/AvailProject) | :satellite:[Explorer](https://telemetry.avail.tools/#/0xd12003ac837853b062aaccca5ce87ac4838c48447e41db4a3dcfb5bf312350c6)

## Update system and install tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget build-essential git jq tar pkg-config libssl-dev liblz4-tool ncdu bashtop clang llvm libudev-dev make protobuf-compiler -y
```

## Create a directory and download a binary file

```bash
mkdir -p $HOME/.avail
cd $HOME/.avail
wget https://github.com/availproject/avail/releases/download/v1.7.2/data-avail-ubuntu-2204-amd64.tar.gz
tar -xvf data-avail-ubuntu-2204-amd64.tar.gz
mv data-avail-ubuntu-2204-amd64 /usr/bin/avail
avail --version
```

## Download json file

```bash
wget -O $HOME/.avail/chainspec.raw.json "https://kate.avail.tools/chainspec.raw.json"
chmod 744 ~/.avail/chainspec.raw.json
```

## Create configuration file

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

```bash
tee /etc/systemd/system/avail.service > /dev/null << EOF
[Unit]
Description=Avail Node
After=network-online.target
StartLimitIntervalSec=0

[Service]
User=$USER
Type=simple
Restart=always
RestartSec=5
LimitNOFILE=65535
ExecStart=/usr/bin/avail \
  --base-path $HOME/.avail/data/ \
  --name $MONIKER \
  --chain $HOME/.avail/chainspec.raw.json \
  --port 30333 \
  --rpc-port 9933 \
  --prometheus-port 9615 \
  --validator

[Install]
WantedBy=multi-user.target
EOF
```

## Start the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable avail
sudo systemctl start avail
sudo systemctl status avail
```

## Check the logs

```bash
sudo journalctl -u avail -f -o cat
```

## Delete node

```bash
sudo systemctl stop avail
sudo systemctl disable avail
sudo rm -rf /etc/systemd/system/avail.service
sudo systemctl daemon-reload
rm /usr/bin/avail
rm -rf $HOME/.avail
```
