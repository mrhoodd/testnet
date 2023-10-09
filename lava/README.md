# Install Guide Lava

## [Website](https://lavanet.xyz/) | [Twitter](https://twitter.com/lavanetxyz) | [Discord](https://discord.gg/5VcqgwMmkA) | :satellite:[Explorer](https://explorer.moonbridge.team/lava-test)

**Chain ID:** lava-testnet-2 | **Latest Version:** v0.24.0 | **Custom Port:** 134

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
LAVA_PORT=134
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
git clone https://github.com/lavanet/lava.git
cd lava
git checkout v0.24.0
make install
lavad version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
lavad config node tcp://localhost:${LAVA_PORT}57
lavad config chain-id lava-testnet-2
lavad config keyring-backend test
lavad init $MONIKER --chain-id lava-testnet-2

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/genesis.json > $HOME/.lava/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/addrbook.json > $HOME/.lava/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.lava/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ulava\"|" $HOME/.lava/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.lava/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.lava/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.lava/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.lava/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.lava/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.lava/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${LAVA_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${LAVA_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${LAVA_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${LAVA_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${LAVA_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${LAVA_PORT}66\"%" $HOME/.lava/config/config.toml
sed -i -e "s%:1317%:${LAVA_PORT}17%g; s%:8080%:${LAVA_PORT}80%g; s%:9090%:${LAVA_PORT}90%g; s%:9091%:${LAVA_PORT}91%g; s%:8545%:${LAVA_PORT}45%g; s%:8546%:${LAVA_PORT}46%g; s%:6065%:${LAVA_PORT}65%g" $HOME/.lava/config/app.toml
```

## Chain configuration

```bash
sed -i \
  -e 's/timeout_commit = ".*"/timeout_commit = "30s"/g' \
  -e 's/timeout_propose = ".*"/timeout_propose = "1s"/g' \
  -e 's/timeout_precommit = ".*"/timeout_precommit = "1s"/g' \
  -e 's/timeout_precommit_delta = ".*"/timeout_precommit_delta = "500ms"/g' \
  -e 's/timeout_prevote = ".*"/timeout_prevote = "1s"/g' \
  -e 's/timeout_prevote_delta = ".*"/timeout_prevote_delta = "500ms"/g' \
  -e 's/timeout_propose_delta = ".*"/timeout_propose_delta = "500ms"/g' \
  -e 's/skip_timeout_commit = ".*"/skip_timeout_commit = false/g' \
  $HOME/.lava/config/config.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/lavad.service > /dev/null <<EOF
[Unit]
Description=Lava Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which lavad) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable lavad
```

## Start service and check the logs

```bash
sudo systemctl start lavad && sudo journalctl -u lavad -f --no-hostname -o cat
```

## Create wallet

```bash
lavad keys add wallet
```

## Check wallet balance

```bash
lavad q bank balances $(lavad keys show wallet -a)
```

## Create validator

```bash
lavad tx staking create-validator \
  --amount 1000000ulava \
  --pubkey $(lavad tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id lava-testnet-2 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0ulava \
  -y
```

## Delete node

```bash
sudo systemctl stop lavad
sudo systemctl disable lavad
sudo rm -rf /etc/systemd/system/lavad.service
sudo systemctl daemon-reload
sudo rm -f $(which lavad) 
sudo rm -rf $HOME/.lava
sudo rm -rf $HOME/lava
```
