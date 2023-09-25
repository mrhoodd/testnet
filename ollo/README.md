# Install Guide Ollo

## [Website](https://linktr.ee/OllOStation) | [Twitter](https://mobile.twitter.com/OllOStation) | [Discord](https://discord.gg/GxBqZ9mSSm) | :satellite:[Explorer](https://explorer.moonbridge.team/ollo-test)

**Chain ID:** ollo-testnet-1 | **Latest Version:** v0.0.1 | **Custom Port:** 132

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
OLLO_PORT=132
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
git clone https://github.com/OLLO-Station/ollo.git
cd ollo
git checkout v0.0.1
make install
ollod version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
ollod config node tcp://localhost:${OLLO_PORT}57
ollod config chain-id ollo-testnet-1
ollod config keyring-backend test
ollod init $MONIKER --chain-id ollo-testnet-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/genesis.json > $HOME/.ollo/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/addrbook.json > $HOME/.ollo/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.ollo/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0utollo\"|" $HOME/.ollo/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.ollo/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.ollo/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.ollo/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.ollo/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.ollo/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.ollo/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${OLLO_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${OLLO_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OLLO_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${OLLO_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${OLLO_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${OLLO_PORT}66\"%" $HOME/.ollo/config/config.toml
sed -i -e "s%:1317%:${OLLO_PORT}17%g; s%:8080%:${OLLO_PORT}80%g; s%:9090%:${OLLO_PORT}90%g; s%:9091%:${OLLO_PORT}91%g; s%:8545%:${OLLO_PORT}45%g; s%:8546%:${OLLO_PORT}46%g; s%:6065%:${OLLO_PORT}65%g" $HOME/.ollo/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/ollod.service > /dev/null <<EOF
[Unit]
Description=Ollo Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which ollod) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable ollod
```

## Start service and check the logs

```bash
sudo systemctl start ollod && sudo journalctl -u ollod -f --no-hostname -o cat
```

## Create wallet

```bash
ollod keys add wallet
```

## Check wallet balance

```bash
ollod q bank balances $(ollod keys show wallet -a)
```

## Create validator

```bash
ollod tx staking create-validator \
  --amount 1000000utollo \
  --pubkey $(ollod tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id ollo-testnet-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0utollo \
  -y
```

## Delete node

```bash
sudo systemctl stop ollod
sudo systemctl disable ollod
sudo rm -rf /etc/systemd/system/ollod.service
sudo systemctl daemon-reload
sudo rm -f $(which ollod) 
sudo rm -rf $HOME/.ollo
sudo rm -rf $HOME/ollo
```
