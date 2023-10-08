
# Install Guide Xion

## [Website](https://xion.burnt.com/) | [Discord](https://discord.com/invite/burnt) | [Twitter](https://twitter.com/burnt_xion) | :satellite:[Explorer](https://explorer.moonbridge.team/xion-test)

## Public endpoints

- API: https://xion-test.api.moonbridge.team
- RPC: https://xion-test.rpc.moonbridge.team

**Chain ID:** xion-testnet-1 | **Latest Version:** v0.3.3 | **Custom Port:** 147

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
XION_PORT=147
```

## Update system and install tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget build-essential git jq tar pkg-config libssl-dev liblz4-tool ncdu bashtop -y
```

## Install GO

```bash
cd $HOME
version="1.20.5"
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
git clone https://github.com/burnt-labs/xion.git xion
cd xion
git checkout v0.3.3
make install
xiond version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
xiond config node tcp://localhost:${XION_PORT}57
xiond config chain-id xion-testnet-1
xiond config keyring-backend test
xiond init $MONIKER --chain-id xion-testnet-1

# Download genesis and addrbook
curl -Ls https://moonbridge.team/snapshots/testnet/xion/genesis.json > $HOME/.xiond/config/genesis.json
curl -Ls https://moonbridge.team/snapshots/testnet/xion/addrbook.json > $HOME/.xiond/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.xiond/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025uxion\"|" $HOME/.xiond/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.xiond/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.xiond/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.xiond/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.xiond/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.xiond/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.xiond/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${XION_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${XION_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${XION_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${XION_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${XION_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${XION_PORT}66\"%" $HOME/.xiond/config/config.toml
sed -i -e "s%:1317%:${XION_PORT}17%g; s%:8080%:${XION_PORT}80%g; s%:9090%:${XION_PORT}90%g; s%:9091%:${XION_PORT}91%g; s%:8545%:${XION_PORT}45%g; s%:8546%:${XION_PORT}46%g; s%:6065%:${XION_PORT}65%g" $HOME/.xiond/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/xiond.service > /dev/null <<EOF
[Unit]
Description=Xion Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.xiond
ExecStart=$(which xiond) start --home $HOME/.xiond
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable xiond
```

## Start service and check the logs

```bash
sudo systemctl start xiond && sudo journalctl -u xiond -f --no-hostname -o cat
```

## Create wallet

```bash
xiond keys add wallet
```

## Recover wallet

```bash
xiond keys add wallet --recover
```

## Check wallet balance

```bash
xiond q bank balances $(xiond keys show wallet -a)
```

## Create validator

```bash
xiond tx staking create-validator \
  --amount 1000000uxion \
  --pubkey $(xiond tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id xion-testnet-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.25 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.2 \
  --gas auto \
  --gas-prices 0.025uxion \
  -y
```

## Delete node

```bash
sudo systemctl stop xiond
sudo systemctl disable xiond
sudo rm -rf /etc/systemd/system/xiond.service
sudo systemctl daemon-reload
sudo rm -f $(which xiond) 
sudo rm -rf $HOME/.xiond
sudo rm -rf $HOME/xion
```
