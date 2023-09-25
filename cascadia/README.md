# Install Guide Cascadia

## [Website](https://www.cascadia.foundation/) | [Twitter](https://twitter.com/CascadiaSystems) | [Discord](https://discord.gg/cascadia) | :satellite:[Explorer](https://explorer.moonbridge.team/cascadia-test)

**Chain ID:** cascadia_6102-1 | **Latest Version:** v0.1.5 | **Custom Port:** 125

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
CASCADIA_PORT=125
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
git clone https://github.com/CascadiaFoundation/cascadia
cd cascadia
git checkout v0.1.5
make install
cascadiad version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
cascadiad config node tcp://localhost:${CASCADIA_PORT}57
cascadiad config chain-id cascadia_6102-1
cascadiad config keyring-backend test
cascadiad init $MONIKER --chain-id cascadia_6102-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Althea/althea-testnet-4/genesis.json > $HOME/.cascadiad/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Althea/althea-testnet-4/addrbook.json > $HOME/.cascadiad/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.cascadiad/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025aCC\"|" $HOME/.cascadiad/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.cascadiad/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.cascadiad/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.cascadiad/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.cascadiad/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.cascadiad/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.cascadiad/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CASCADIA_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${CASCADIA_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CASCADIA_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CASCADIA_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${CASCADIA_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CASCADIA_PORT}66\"%" $HOME/.cascadiad/config/config.toml
sed -i -e "s%:1317%:${CASCADIA_PORT}17%g; s%:8080%:${CASCADIA_PORT}80%g; s%:9090%:${CASCADIA_PORT}90%g; s%:9091%:${CASCADIA_PORT}91%g; s%:8545%:${CASCADIA_PORT}45%g; s%:8546%:${CASCADIA_PORT}46%g; s%:6065%:${CASCADIA_PORT}65%g" $HOME/.cascadiad/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/cascadiad.service > /dev/null <<EOF
[Unit]
Description=Cascadia Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cascadiad) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable cascadiad
```

## Start service and check the logs

```bash
sudo systemctl start cascadiad && sudo journalctl -u cascadiad -f --no-hostname -o cat
```

## Create wallet

```bash
cascadiad keys add wallet
```

## Check wallet balance

```bash
cascadiad q bank balances $(cascadiad keys show wallet -a)
```

## Create validator

```bash
cascadiad tx staking create-validator \
  --amount 1000000aCC \
  --pubkey $(cascadiad tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id cascadia_6102-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 7aCC \
  -y
```

## Firewall security

```bash
sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${CASCADIA_PORT}56/tcp
sudo ufw enable
```

## Delete node

```bash
sudo systemctl stop cascadiad
sudo systemctl disable cascadiad
sudo rm -rf /etc/systemd/system/cascadiad.service
sudo systemctl daemon-reload
sudo rm -f $(which cascadiad) 
sudo rm -rf $HOME/.cascadiad
sudo rm -rf $HOME/cascadia
```
