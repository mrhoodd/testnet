
# Install Guide Elys

## [Website](https://elys.network/) | [Twitter](https://twitter.com/elys_network) | [Discord](https://discord.gg/R9Gr6Vh7vC) | :satellite:[Explorer](https://explorer.moonbridge.team/elys-test)

## Public endpoints

- API: https://elys-test.api.moonbridge.team
- RPC: https://elys-test.rpc.moonbridge.team

**Chain ID:** elystestnet-1 | **Latest Version:** v0.12.0 | **Custom Port:** 146

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
ELYS_PORT=146
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
git clone https://github.com/elys-network/elys.git
cd elys
git checkout v0.12.0
make install
elysd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
elysd config node tcp://localhost:${ELYS_PORT}57
elysd config chain-id elystestnet-1
elysd config keyring-backend test
elysd init $MONIKER --chain-id elystestnet-1

# Download genesis and addrbook
curl -Ls https://moonbridge.team/snapshots/testnet/elys/genesis.json > $HOME/.elys/config/genesis.json
curl -Ls https://moonbridge.team/snapshots/testnet/elys/addrbook.json > $HOME/.elys/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.elys/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0018ibc/2180E84E20F5679FCC760D8C165B60F42065DEF7F46A72B447CFF1B7DC6C0A65,0.00025ibc/27394FB092D2ECCD56123C74F36E4C1F926001CEADA9CA97EA622B25F41E5EB2,0.00025uelys\"|" $HOME/.elys/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.elys/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.elys/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.elys/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.elys/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.elys/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.elys/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ELYS_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${ELYS_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ELYS_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ELYS_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ELYS_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ELYS_PORT}66\"%" $HOME/.elys/config/config.toml
sed -i -e "s%:1317%:${ELYS_PORT}17%g; s%:8080%:${ELYS_PORT}80%g; s%:9090%:${ELYS_PORT}90%g; s%:9091%:${ELYS_PORT}91%g; s%:8545%:${ELYS_PORT}45%g; s%:8546%:${ELYS_PORT}46%g; s%:6065%:${ELYS_PORT}65%g" $HOME/.elys/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/elysd.service > /dev/null <<EOF
[Unit]
Description=Elys Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.elys
ExecStart=$(which elysd) start --home $HOME/.elys
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable elysd
```

## Start service and check the logs

```bash
sudo systemctl start elysd && sudo journalctl -u elysd -f --no-hostname -o cat
```

## Create wallet

```bash
elysd keys add wallet
```

## Recover wallet

```bash
elysd keys add wallet --recover
```

## Check wallet balance

```bash
elysd q bank balances $(elysd keys show wallet -a)
```

## Create validator

```bash
elysd tx staking create-validator \
  --amount 1000000uelys \
  --pubkey $(elysd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id elystestnet-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.00025uelys \
  -y
```

## Delete node

```bash
sudo systemctl stop elysd
sudo systemctl disable elysd
sudo rm -rf /etc/systemd/system/elysd.service
sudo systemctl daemon-reload
sudo rm -f $(which elysd) 
sudo rm -rf $HOME/.elys
sudo rm -rf $HOME/elys
```
