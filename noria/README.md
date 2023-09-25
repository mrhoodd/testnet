# Install Guide Noria

## [Website](https://noria.network/) | [Twitter](https://twitter.com/NoriaNetwork) | [Discord](https://discord.gg/pseAWBQ6EZ) | :satellite:[Explorer](https://explorer.moonbridge.team/noria-test)

**Chain ID:** oasis-3 | **Latest Version:** v1.3.0 | **Custom Port:** 120

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
NORIA_PORT=120
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
git clone https://github.com/noria-net/noria.git
cd noria
git checkout v1.3.0
make install
noriad version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
noriad config node tcp://localhost:${NORIA_PORT}57
noriad config chain-id oasis-3
noriad config keyring-backend test
noriad init $MONIKER --chain-id oasis-3

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Althea/althea-testnet-4/genesis.json > $HOME/.noria/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Althea/althea-testnet-4/addrbook.json > $HOME/.noria/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.noria/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0025ucrd\"|" $HOME/.noria/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.noria/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.noria/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.noria/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.noria/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.noria/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.noria/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${NORIA_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${NORIA_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${NORIA_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${NORIA_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${NORIA_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${NORIA_PORT}66\"%" $HOME/.noria/config/config.toml
sed -i -e "s%:1317%:${NORIA_PORT}17%g; s%:8080%:${NORIA_PORT}80%g; s%:9090%:${NORIA_PORT}90%g; s%:9091%:${NORIA_PORT}91%g; s%:8545%:${NORIA_PORT}45%g; s%:8546%:${NORIA_PORT}46%g; s%:6065%:${NORIA_PORT}65%g" $HOME/.noria/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/noriad.service > /dev/null <<EOF
[Unit]
Description=Noria Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noriad) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable noriad
```

## Start service and check the logs

```bash
sudo systemctl start noriad && sudo journalctl -u noriad -f --no-hostname -o cat
```

## Create wallet

```bash
noriad keys add wallet
```

## Check wallet balance

```bash
noriad q bank balances $(noriad keys show wallet -a)
```

## Create validator

```bash
noriad tx staking create-validator \
  --amount 1000000unoria \
  --pubkey $(noriad tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id oasis-3 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.0025ucrd \
  -y
```

## Firewall security

```bash
sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${NORIA_PORT}56/tcp
sudo ufw enable
```

## Delete node

```bash
sudo systemctl stop noriad
sudo systemctl disable noriad
sudo rm -rf /etc/systemd/system/noriad.service
sudo systemctl daemon-reload
sudo rm -f $(which noriad) 
sudo rm -rf $HOME/.noria
sudo rm -rf $HOME/noria
```
