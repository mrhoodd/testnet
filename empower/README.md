# Install Guide Empower
  
## [Website](https://www.empowerchain.io/) | [Discord](https://discord.gg/DNB4z8EZDx) | [Twitter](https://twitter.com/empowerchain_io) | :satellite:[Explorer](https://explorer.moonbridge.team/empower-test)

**Chain ID:** circulus-1 | **Latest Version:** v1.0.0-rc3 | **Custom Port:** 126

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
EMPOWER_PORT=126
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
git clone https://github.com/EmpowerPlastic/empowerchain
cd empowerchain
git checkout v1.0.0-rc3
cd chain
make install
empowerd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
empowerd config node tcp://localhost:${EMPOWER_PORT}57
empowerd config chain-id circulus-1
empowerd config keyring-backend test
empowerd init $MONIKER --chain-id circulus-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/genesis.json > $HOME/.empowerchain/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/addrbook.json > $HOME/.empowerchain/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.empowerchain/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0umpwr\"|" $HOME/.empowerchain/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.empowerchain/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.empowerchain/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.empowerchain/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.empowerchain/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.empowerchain/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.empowerchain/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${EMPOWER_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${EMPOWER_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${EMPOWER_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${EMPOWER_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${EMPOWER_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${EMPOWER_PORT}66\"%" $HOME/.empowerchain/config/config.toml
sed -i -e "s%:1317%:${EMPOWER_PORT}17%g; s%:8080%:${EMPOWER_PORT}80%g; s%:9090%:${EMPOWER_PORT}90%g; s%:9091%:${EMPOWER_PORT}91%g; s%:8545%:${EMPOWER_PORT}45%g; s%:8546%:${EMPOWER_PORT}46%g; s%:6065%:${EMPOWER_PORT}65%g" $HOME/.empowerchain/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/empowerd.service > /dev/null <<EOF
[Unit]
Description=Empower Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which empowerd) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable empowerd
```

## Start service and check the logs

```bash
sudo systemctl start empowerd && sudo journalctl -u empowerd -f --no-hostname -o cat
```

## Create wallet

```bash
empowerd keys add wallet
```

## Check wallet balance

```bash
empowerd q bank balances $(empowerd keys show wallet -a)
```

## Create validator

```bash
empowerd tx staking create-validator \
  --amount 1000000umpwr \
  --pubkey $(empowerd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id circulus-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0umpwr \
  -y
```

## Delete node

```bash
sudo systemctl stop empowerd
sudo systemctl disable empowerd
sudo rm -rf /etc/systemd/system/empowerd.service
sudo systemctl daemon-reload
sudo rm -f $(which empowerd) 
sudo rm -rf $HOME/.empowerchain
sudo rm -rf $HOME/empowerchain
```
