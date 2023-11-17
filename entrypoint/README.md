# Install Guide Entrypoint

## [Website](https://entrypoint.zone/) | [Discord](https://discord.gg/6Ec9jDwVnB) | [Twitter](https://twitter.com/entrypointzone) | :satellite:[Explorer](https://explorer.moonbridge.team/entrypoint-test)

## Public Endpoints

- API: <https://entrypoint-test.api.moonbridge.team>
- RPC: <https://entrypoint-test.rpc.moonbridge.team>
- gRPC: <https://entrypoint-test.grpc.moonbridge.team>

**Chain ID:** entrypoint-pubtest-2 | **Latest Version:** v1.2.0 | **Custom Port:** 129

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
ENTRYPOINT_PORT=129
```

## Update system and install tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget build-essential git jq tar pkg-config libssl-dev liblz4-tool ncdu bashtop -y
```

## Install GO

```bash
cd $HOME
version="1.21.3"
wget "https://golang.org/dl/go$version.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$version.linux-amd64.tar.gz"
rm "go$version.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Download and install

```bash
cd $HOME
wget https://snapshots.moonbridge.team/testnet/entrypoint/entrypointd
chmod +x entrypointd
mv entrypointd $HOME/go/bin/
entrypointd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
entrypointd config node tcp://localhost:${ENTRYPOINT_PORT}57
entrypointd config chain-id entrypoint-pubtest-2
entrypointd config keyring-backend test
entrypointd init $MONIKER --chain-id entrypoint-pubtest-2

# Download genesis and addrbook
curl -Ls https://snapshots.moonbridge.team/testnet/entrypoint/genesis.json > $HOME/.entrypoint/config/genesis.json
curl -Ls https://snapshots.moonbridge.team/testnet/entrypoint/addrbook.json > $HOME/.entrypoint/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.entrypoint/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.01ibc/8A138BC76D0FB2665F8937EC2BF01B9F6A714F6127221A0E155106A45E09BCC5\"|" $HOME/.entrypoint/config/app.toml

# Setting pruning
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.entrypoint/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.entrypoint/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.entrypoint/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.entrypoint/config/app.toml

# Disable indexer
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.entrypoint/config/config.toml

# Enable Prometheus
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.entrypoint/config/config.toml

# Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ENTRYPOINT_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${ENTRYPOINT_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ENTRYPOINT_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ENTRYPOINT_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ENTRYPOINT_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ENTRYPOINT_PORT}66\"%" $HOME/.entrypoint/config/config.toml
sed -i -e "s%:1317%:${ENTRYPOINT_PORT}17%g; s%:8080%:${ENTRYPOINT_PORT}80%g; s%:9090%:${ENTRYPOINT_PORT}90%g; s%:9091%:${ENTRYPOINT_PORT}91%g; s%:8545%:${ENTRYPOINT_PORT}45%g; s%:8546%:${ENTRYPOINT_PORT}46%g; s%:6065%:${ENTRYPOINT_PORT}65%g" $HOME/.entrypoint/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/entrypointd.service > /dev/null <<EOF
[Unit]
Description=Entrypoint Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.entrypoint
ExecStart=$(which entrypointd) start --home $HOME/.entrypoint
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable entrypointd
```

## Start service and check the logs

```bash
sudo systemctl start entrypointd && sudo journalctl -u entrypointd -f --no-hostname -o cat
```

## Create wallet

```bash
entrypointd keys add wallet
```

## Check wallet balance

```bash
entrypointd q bank balances $(entrypointd keys show wallet -a)
```

## Create validator

```bash
entrypointd tx staking create-validator \
  --amount 1000000uentry \
  --pubkey $(entrypointd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id entrypoint-pubtest-2 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.01ibc/8A138BC76D0FB2665F8937EC2BF01B9F6A714F6127221A0E155106A45E09BCC5 \
  -y
```

## Delete node

```bash
sudo systemctl stop entrypointd
sudo systemctl disable entrypointd
sudo rm -rf /etc/systemd/system/entrypointd.service
sudo systemctl daemon-reload
sudo rm -f $(which entrypointd) 
sudo rm -rf $HOME/.entrypoint
```
