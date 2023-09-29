# Install Guide Structs

## [Website](https://www.playstructs.com/) | [Twitter](https://twitter.com/PlayStructs) | [Discord](https://discord.gg/UQ5EXTfjpa) | :satellite:[Explorer](https://explorer.moonbridge.team/structs-test)

## Public endpoints
- API: https://structs-test.api.moonbridge.team
- RPC: https://structs-test.rpc.moonbridge.team

**Chain ID:** structstestnet-74 | **Latest Version:** 0.1.0-beta | **Custom Port:** 144

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
STRUCTS_PORT=144
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
curl https://get.ignite.com/cli! | bash
ignite version
ignite network chain install 74
structsd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
structsd config node tcp://localhost:${STRUCTS_PORT}57
structsd config chain-id structstestnet-74
structsd config keyring-backend test
structsd init $MONIKER --chain-id structstestnet-74

# Download genesis and addrbook
curl -Ls https://moonbridge.team/snapshots/testnet/structs/genesis.json > $HOME/.structs/config/genesis.json
curl -Ls https://moonbridge.team/snapshots/testnet/structs/addrbook.json > $HOME/.structs/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.structs/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0alpha\"|" $HOME/.structs/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.structs/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.structs/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.structs/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.structs/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.structs/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.structs/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${STRUCTS_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${STRUCTS_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${STRUCTS_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${STRUCTS_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${STRUCTS_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${STRUCTS_PORT}66\"%" $HOME/.structs/config/config.toml
sed -i -e "s%:1317%:${STRUCTS_PORT}17%g; s%:8080%:${STRUCTS_PORT}80%g; s%:9090%:${STRUCTS_PORT}90%g; s%:9091%:${STRUCTS_PORT}91%g; s%:8545%:${STRUCTS_PORT}45%g; s%:8546%:${STRUCTS_PORT}46%g; s%:6065%:${STRUCTS_PORT}65%g" $HOME/.structs/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/structsd.service > /dev/null <<EOF
[Unit]
Description=Structs Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.structs
ExecStart=$(which structsd) start --home $HOME/.structs
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable structsd
```

## Start service and check the logs

```bash
sudo systemctl start structsd && sudo journalctl -u structsd -f --no-hostname -o cat
```

## Create wallet

```bash
structsd keys add wallet
```

## Recover wallet

```bash
structsd keys add wallet --recover
```

## Check wallet balance

```bash
structsd q bank balances $(structsd keys show wallet -a)
```

## Create validator

```bash
structsd tx staking create-validator \
  --amount 1000000alpha \
  --pubkey $(structsd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id structstestnet-74 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.0alpha \
  -y
```

## Delete node

```bash
sudo systemctl stop structsd
sudo systemctl disable structsd
sudo rm -rf /etc/systemd/system/structsd.service
sudo systemctl daemon-reload
sudo rm -f $(which structsd) 
sudo rm -rf $HOME/.structs
sudo rm -rf $HOME/source
```
