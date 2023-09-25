# Install Guide Andromeda

## [Website](https://andromedaprotocol.io/) | [Twitter](https://twitter.com/andromedaprot) | [Discord](https://discord.gg/GBd6buKYyZ) | :satellite:[Explorer](https://explorer.moonbridge.team/Andromeda-t)

**Chain ID:** galileo-3 | **Latest Version:** galileo-3-v1.1.0-beta1 | **Custom Port:** 127

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
ANDROMEDA_PORT=127
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
git clone https://github.com/andromedaprotocol/andromedad.git
cd andromedad
git checkout galileo-3-v1.1.0-beta1
make install
andromedad version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
andromedad config node tcp://localhost:${ANDROMEDA_PORT}57
andromedad config chain-id galileo-3
andromedad config keyring-backend test
andromedad init $MONIKER --chain-id galileo-3

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Andromeda/genesis.json > $HOME/.andromedad/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Andromeda/addrbook.json > $HOME/.andromedad/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.andromedad/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001uandr\"|" $HOME/.andromedad/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.andromedad/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.andromedad/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.andromedad/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.andromedad/config/app.toml

# Disable indexer
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.andromedad/config/config.toml

# Enable Prometheus
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.andromedad/config/config.toml

# Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ANDROMEDA_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${ANDROMEDA_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ANDROMEDA_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ANDROMEDA_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ANDROMEDA_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ANDROMEDA_PORT}66\"%" $HOME/.andromedad/config/config.toml
sed -i -e "s%:1317%:${ANDROMEDA_PORT}17%g; s%:8080%:${ANDROMEDA_PORT}80%g; s%:9090%:${ANDROMEDA_PORT}90%g; s%:9091%:${ANDROMEDA_PORT}91%g; s%:8545%:${ANDROMEDA_PORT}45%g; s%:8546%:${ANDROMEDA_PORT}46%g; s%:6065%:${ANDROMEDA_PORT}65%g" $HOME/.andromedad/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/andromedad.service > /dev/null << EOF
[Unit]
Description=Andromeda Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which andromedad) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable andromedad
```

## Start service and check the logs

```bash
sudo systemctl start andromedad && sudo journalctl -u andromedad -f --no-hostname -o cat
```

## Create wallet

```bash
andromedad keys add wallet
```

## Check wallet balance

```bash
andromedad q bank balances $(andromedad keys show wallet -a)
```

## Create validator

```bash
andromedad tx staking create-validator \
  --amount 1000000uandr \
  --pubkey $(andromedad tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id galileo-3 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.5 \
  --gas auto \
  --gas-prices 0.0001uandr \
  -y
```

## Firewall security

```bash
sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${ANDROMEDA_PORT}56/tcp
sudo ufw enable
```

## Delete node

```bash
cd $HOME
sudo systemctl stop andromedad
sudo systemctl disable andromedad
sudo rm /etc/systemd/system/andromedad.service
sudo systemctl daemon-reload
rm -f $(which andromedad)
rm -rf $HOME/.andromedad
rm -rf $HOME/andromedad
```
