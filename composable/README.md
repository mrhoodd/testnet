# Install Guide Composable

## [Website](https://www.composable.finance/) | [Discord](https://discord.gg/composable) | [Twitter](https://twitter.com/ComposableFin) | :satellite:[Explorer](https://explorer.moonbridge.team/composable-test)

**Chain ID:** banksy-testnet-4 | **Latest Version:** v6.2.3-testnet | **Custom Port:** 122

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
BANKSY_PORT=122
```

## Update system and install tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget build-essential git jq tar pkg-config libssl-dev liblz4-tool ncdu bashtop -y
```

## Install GO

```bash
cd $HOME
version="1.21.4"
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
git clone https://github.com/notional-labs/composable-centauri
cd composable-centauri
git checkout v6.2.3-testnet
make install
layerd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
layerd config node tcp://localhost:${BANKSY_PORT}57
layerd config chain-id banksy-testnet-4
layerd config keyring-backend test
layerd init $MONIKER --chain-id banksy-testnet-4

# Download genesis and addrbook
curl -Ls https://snapshots.moonbridge.team/testnet/composable/genesis.json > $HOME/.banksy/config/genesis.json
curl -Ls https://snapshots.moonbridge.team/testnet/composable/addrbook.json > $HOME/.banksy/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.banksy/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ppica\"|" $HOME/.banksy/config/app.toml

# Setting pruning
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.banksy/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.banksy/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.banksy/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.banksy/config/app.toml

# Disable indexer
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.banksy/config/config.toml

# Enable Prometheus
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.banksy/config/config.toml

# Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${BANKSY_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${BANKSY_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${BANKSY_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${BANKSY_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${BANKSY_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${BANKSY_PORT}66\"%" $HOME/.banksy/config/config.toml
sed -i -e "s%:1317%:${BANKSY_PORT}17%g; s%:8080%:${BANKSY_PORT}80%g; s%:9090%:${BANKSY_PORT}90%g; s%:9091%:${BANKSY_PORT}91%g; s%:8545%:${BANKSY_PORT}45%g; s%:8546%:${BANKSY_PORT}46%g; s%:6065%:${BANKSY_PORT}65%g" $HOME/.banksy/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/layerd.service > /dev/null <<EOF
[Unit]
Description=Composable Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.banksy
ExecStart=$(which layerd) start --home $HOME/.banksy
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable layerd
```

## Start service and check the logs

```bash
sudo systemctl start layerd && sudo journalctl -u layerd -f --no-hostname -o cat
```

## Create wallet

```bash
layerd keys add wallet
```

## Recover wallet

```bash
layerd keys add wallet --recover
```

## Check wallet balance

```bash
layerd q bank balances $(layerd keys show wallet -a)
```

## Create validator

```bash
layerd tx staking create-validator \
  --amount 1000000000000ppica \
  --pubkey $(layerd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id banksy-testnet-4 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0ppica \
  -y
```

## Firewall security

```bash
sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${BANKSY_PORT}56/tcp
sudo ufw enable
```

## Delete node

```bash
sudo systemctl stop layerd
sudo systemctl disable layerd
sudo rm -rf /etc/systemd/system/layerd.service
sudo systemctl daemon-reload
sudo rm -f $(which layerd) 
sudo rm -rf $HOME/.banksy
sudo rm -rf $HOME/composable-centauri
```
