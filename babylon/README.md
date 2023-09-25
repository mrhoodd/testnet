# Install Guide Babylon

## [Website](https://www.babylonchain.io/) | [Discord](http://discord.gg/babylonchain) | [Twitter](https://twitter.com/babylon_chain) | :satellite:[Explorer](https://explorer.moonbridge.team/babylon-test)

**Chain ID:** bbn-test-2 | **Latest Version:** v0.7.2 | **Custom Port:** 128

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
BABYLON_PORT=128
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
git clone https://github.com/babylonchain/babylon
cd babylon
git checkout v0.7.2
make install
babylond version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
babylond config node tcp://localhost:${BABYLON_PORT}57
babylond config chain-id bbn-test-2
babylond config keyring-backend test
babylond init $MONIKER --chain-id bbn-test-2

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/genesis.json > $HOME/.babylond/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/addrbook.json > $HOME/.babylond/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.babylond/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.00001ubbn\"|" $HOME/.babylond/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.babylond/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.babylond/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.babylond/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.babylond/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.babylond/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.babylond/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${BABYLON_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${BABYLON_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${BABYLON_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${BABYLON_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${BABYLON_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${BABYLON_PORT}66\"%" $HOME/.babylond/config/config.toml
sed -i -e "s%:1317%:${BABYLON_PORT}17%g; s%:8080%:${BABYLON_PORT}80%g; s%:9090%:${BABYLON_PORT}90%g; s%:9091%:${BABYLON_PORT}91%g; s%:8545%:${BABYLON_PORT}45%g; s%:8546%:${BABYLON_PORT}46%g; s%:6065%:${BABYLON_PORT}65%g" $HOME/.babylond/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/babylond.service > /dev/null <<EOF
[Unit]
Description=Babylon Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which babylond) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable babylond
```

## Start service and check the logs

```bash
sudo systemctl start babylond && sudo journalctl -u babylond -f --no-hostname -o cat
```

## Create wallet

```bash
babylond keys add wallet
```

## Check wallet balance

```bash
babylond q bank balances $(babylond keys show wallet -a)
```

## Create a BLS key

```bash
babylond create-bls-key $(babylond keys show wallet -a)
```

## Restart node

```bash
sudo systemctl restart babylond
```

## Modify config

```bash
sed -i -e "s|^key-name *=.*|key-name = \"wallet\"|" $HOME/.babylond/config/app.toml
sed -i -e "s|^timeout_commit *=.*|timeout_commit = \"10s\"|" $HOME/.babylond/config/config.toml
```

## Create validator

```bash
babylond tx staking create-validator \
  --amount 1000000ubbn \
  --pubkey $(babylond tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id bbn-test-2 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.00001ubbn \
  -y
```

## Delete node

```bash
sudo systemctl stop babylond
sudo systemctl disable babylond
sudo rm -rf /etc/systemd/system/babylond.service
sudo systemctl daemon-reload
sudo rm -f $(which babylond) 
sudo rm -rf $HOME/.babylond
sudo rm -rf $HOME/babylon
```
