# Install Guide Nibiru

## [Website](https://nibiru.fi/) | [Twitter](https://twitter.com/NibiruChain) | [Discord](https://discord.gg/bFamKuv9) | :satellite:[Explorer](https://explorer.moonbridge.team/nibiru-test)

**Chain ID:** nibiru-itn-2 | **Latest Version:** v0.21.9 | **Custom Port:** 136

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
NIBIRU_PORT=136
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
git clone https://github.com/NibiruChain/nibiru
cd nibiru
git checkout v0.21.9
make install
nibid version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
nibid config node tcp://localhost:${NIBIRU_PORT}57
nibid config chain-id nibiru-itn-2
nibid config keyring-backend test
nibid init $MONIKER --chain-id nibiru-itn-2

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/genesis.json > $HOME/.nibid/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/addrbook.json > $HOME/.nibid/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.nibid/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025unibi\"|" $HOME/.nibid/config/app.toml

# Setting pruning
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.nibid/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.nibid/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.nibid/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.nibid/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.nibid/config/config.toml

# Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.nibid/config/config.toml

# Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${NIBIRU_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${NIBIRU_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${NIBIRU_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${NIBIRU_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${NIBIRU_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${NIBIRU_PORT}66\"%" $HOME/.nibid/config/config.toml
sed -i -e "s%:1317%:${NIBIRU_PORT}17%g; s%:8080%:${NIBIRU_PORT}80%g; s%:9090%:${NIBIRU_PORT}90%g; s%:9091%:${NIBIRU_PORT}91%g; s%:8545%:${NIBIRU_PORT}45%g; s%:8546%:${NIBIRU_PORT}46%g; s%:6065%:${NIBIRU_PORT}65%g" $HOME/.nibid/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=Nibiru Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable nibid
```

## Start service and check the logs

```bash
sudo systemctl start nibid && sudo journalctl -u nibid -f --no-hostname -o cat
```

## Create wallet

```bash
nibid keys add wallet
```

## Check wallet balance

```bash
nibid q bank balances $(nibid keys show wallet -a)
```

## Create validator

```bash
nibid tx staking create-validator \
  --amount 1000000unibi \
  --pubkey $(nibid tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id nibiru-itn-2 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.025unibi \
  -y
```

## Install Price feeder

```bash
curl -s https://get.nibiru.fi/pricefeeder@v0.21.3! | bash
```

Create wallet for pricefeeder

```bash
nibid keys add pricefeeder-wallet
```

Set variables for price feeder

```bash
export CHAIN_ID="nibiru-itn-2"
export GRPC_ENDPOINT="localhost:13690"
export WEBSOCKET_ENDPOINT="ws://localhost:13657/websocket"
export EXCHANGE_SYMBOLS_MAP='{"bitfinex":{"ubtc:unusd":"tBTCUSD","ubtc:uusd":"tBTCUSD","ueth:unusd":"tETHUSD","ueth:uusd":"tETHUSD","uusdc:uusd":"tUDCUSD","uusdc:unusd":"tUDCUSD"},"coingecko":{"ubtc:uusd":"bitcoin","ubtc:unusd":"bitcoin","ueth:uusd":"ethereum","ueth:unusd":"ethereum","uusdt:uusd":"tether","uusdt:unusd":"tether","uusdc:uusd":"usd-coin","uusdc:unusd":"usd-coin","uatom:uusd":"cosmos","uatom:unusd":"cosmos","ubnb:uusd":"binancecoin","ubnb:unusd":"binancecoin","uavax:uusd":"avalanche-2","uavax:unusd":"avalanche-2","usol:uusd":"solana","usol:unusd":"solana","uada:uusd":"cardano","uada:unusd":"cardano"}}'
export FEEDER_MNEMONIC="<YOUR_MNEMONIC_HERE>"
export VALIDATOR_ADDRESS=$(nibid keys show wallet --bech val -a)
```

## Create service price feeder

```bash
sudo tee /etc/systemd/system/pricefeeder.service<<EOF
[Unit]
Description=Nibiru Pricefeeder
After=network-online.target

[Service]
Type=exec
User=$USER
Group=$USER
ExecStart=/usr/local/bin/pricefeeder
Restart=on-failure
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGTERM
PermissionsStartOnly=true
LimitNOFILE=65535
Environment=CHAIN_ID='$CHAIN_ID'
Environment=GRPC_ENDPOINT='$GRPC_ENDPOINT'
Environment=WEBSOCKET_ENDPOINT='$WEBSOCKET_ENDPOINT'
Environment=EXCHANGE_SYMBOLS_MAP='$EXCHANGE_SYMBOLS_MAP'
Environment=FEEDER_MNEMONIC='$FEEDER_MNEMONIC'
Environment=VALIDATOR_ADDRESS='$VALIDATOR_ADDRESS'

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable pricefeeder
```

## Start service and check the logs

```bash
sudo systemctl start pricefeeder && journalctl -fu pricefeeder -o cat
```

## Delete node

```bash
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm -rf /etc/systemd/system/nibid.service
sudo systemctl daemon-reload
sudo rm -f $(which nibid) 
sudo rm -rf $HOME/.nibid
sudo rm -rf $HOME/nibiru
```
