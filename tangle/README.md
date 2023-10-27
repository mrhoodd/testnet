# Install Guide Tangle

## [Website](https://www.tangle.tools/) | [Discord](https://discord.com/invite/cv8EfJu3Tn) | [Twitter](https://twitter.com/webbprotocol) | :satellite:[Explorer](https://telemetry.polkadot.io/#list/0xea63e6ac7da8699520af7fb540470d63e48eccb33f7273d2e21a935685bf1320)

## Update system and install tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget build-essential git jq tar pkg-config libssl-dev liblz4-tool ncdu bashtop clang llvm libudev-dev make protobuf-compiler -y
```

## Create a directory and download a binary file

```bash
mkdir -p $HOME/.tangle
cd $HOME/.tangle
wget -O tangle https://github.com/webb-tools/tangle/releases/download/v0.4.9/tangle-standalone-linux-amd64
chmod 744 tangle
mv tangle /usr/bin/
tangle --version
```

## Create configuration file

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

```bash
tee /etc/systemd/system/tangle.service > /dev/null << EOF
[Unit]
Description=Tangle Validator Node
After=network-online.target
StartLimitIntervalSec=0

[Service]
User=$USER
Restart=always
RestartSec=5
LimitNOFILE=65535
ExecStart=/usr/bin/tangle \
  --base-path $HOME/.tangle/data/ \
  --name $MONIKER \
  --chain tangle-testnet \
  --node-key-file "$HOME/.tangle/node-key" \
  --port 30333 \
  --rpc-port 9933 \
  --prometheus-port 9615 \
  --auto-insert-keys \
  --validator \
  --no-mdns \
  --telemetry-url "wss://telemetry.polkadot.io/submit 0"

[Install]
WantedBy=multi-user.target
EOF
```

## Start the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable tangle
sudo systemctl start tangle
```

## Check the logs

```bash
sudo journalctl -u tangle -f -o cat
```

## Delete node

```bash
sudo systemctl stop tangle
sudo systemctl disable tangle
sudo rm -rf /etc/systemd/system/tangle.service
sudo systemctl daemon-reload
rm /usr/bin/tangle
rm -rf $HOME/.tangle
```
