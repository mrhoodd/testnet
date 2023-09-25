# Install Guide Dymension

## [Website](https://www.dymension.xyz/) | [Discord](https://t.co/jY0E3m5vwu) | [Twitter](https://twitter.com/dYmensionXYZ) | :satellite:[Explorer](https://explorer.moonbridge.team/dymension-test)

**Chain ID:** froopyland_100-1 | **Latest Version:** v1.0.2-beta | **Custom Port:** 137

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
DYMENSION_PORT=137
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
git clone https://github.com/dymensionxyz/dymension.git
cd dymension
git checkout v1.0.2-beta
make install
dymd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
dymd config node tcp://localhost:${DYMENSION_PORT}57
dymd config chain-id froopyland_100-1
dymd config keyring-backend test
dymd init $MONIKER --chain-id froopyland_100-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/genesis.json > $HOME/.dymension/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/addrbook.json > $HOME/.dymension/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.dymension/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025udym,0.025uatom\"|" $HOME/.dymension/config/app.toml

# Setting pruning
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.dymension/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.dymension/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.dymension/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.dymension/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.dymension/config/config.toml

# Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.dymension/config/config.toml

# Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${DYMENSION_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${DYMENSION_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${DYMENSION_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${DYMENSION_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${DYMENSION_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${DYMENSION_PORT}66\"%" $HOME/.dymension/config/config.toml
sed -i -e "s%:1317%:${DYMENSION_PORT}17%g; s%:8080%:${DYMENSION_PORT}80%g; s%:9090%:${DYMENSION_PORT}90%g; s%:9091%:${DYMENSION_PORT}91%g; s%:8545%:${DYMENSION_PORT}45%g; s%:8546%:${DYMENSION_PORT}46%g; s%:6065%:${DYMENSION_PORT}65%g" $HOME/.dymension/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/dymd.service > /dev/null <<EOF
[Unit]
Description=Dymension Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which dymd) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable dymd
```

## Start service and check the logs

```bash
sudo systemctl start dymd && sudo journalctl -u dymd -f --no-hostname -o cat
```

## Create wallet

```bash
dymd keys add wallet
```

## Check wallet balance

```bash
dymd q bank balances $(dymd keys show wallet -a)
```

## Create validator

```bash
dymd tx staking create-validator \
  --amount 1000000udym \
  --pubkey $(dymd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id froopyland_100-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.025udym \
  -y
```

## Delete node

```bash
sudo systemctl stop dymd
sudo systemctl disable dymd
sudo rm -rf /etc/systemd/system/dymd.service
sudo systemctl daemon-reload
sudo rm -f $(which dymd) 
sudo rm -rf $HOME/.dymension
sudo rm -rf $HOME/dymension
```
