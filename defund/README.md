# Install Guide Defund

## [Website](https://www.defund.app/) | [Twitter](https://twitter.com/defund_finance) | [Discord](https://discord.gg/AcMx4zBj) | [App](https://hq.defund.app/#/) | [Explorer](https://explorer.moonbridge.team/defund-test)

**Chain ID:** orbit-alpha-1 | **Latest Version:** v0.2.6 | **Custom Port:** 133

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
DEFUND_PORT=133
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
git clone https://github.com/defund-labs/defund.git
cd defund
git checkout v0.2.6
make install
defundd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
defundd config node tcp://localhost:${DEFUND_PORT}57
defundd config chain-id orbit-alpha-1
defundd config keyring-backend test
defundd init $MONIKER --chain-id orbit-alpha-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/genesis.json > $HOME/.defund/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/addrbook.json > $HOME/.defund/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.defund/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ufetf\"|" $HOME/.defund/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.defund/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.defund/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.defund/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.defund/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.defund/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.defund/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${DEFUND_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${DEFUND_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${DEFUND_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${DEFUND_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${DEFUND_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${DEFUND_PORT}66\"%" $HOME/.defund/config/config.toml
sed -i -e "s%:1317%:${DEFUND_PORT}17%g; s%:8080%:${DEFUND_PORT}80%g; s%:9090%:${DEFUND_PORT}90%g; s%:9091%:${DEFUND_PORT}91%g; s%:8545%:${DEFUND_PORT}45%g; s%:8546%:${DEFUND_PORT}46%g; s%:6065%:${DEFUND_PORT}65%g" $HOME/.defund/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/defundd.service > /dev/null <<EOF
[Unit]
Description=Defund Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which defundd) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable defundd
```

## Start service and check the logs

```bash
sudo systemctl start defundd && sudo journalctl -u defundd -f --no-hostname -o cat
```

## Create wallet

```bash
defundd keys add wallet
```

## Check wallet balance

```bash
defundd q bank balances $(defundd keys show wallet -a)
```

## Create validator

```bash
defundd tx staking create-validator \
  --amount 1000000ufetf \
  --pubkey $(defundd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id orbit-alpha-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0ufetf \
  -y
```

## Delete node

```bash
sudo systemctl stop defundd
sudo systemctl disable defundd
sudo rm -rf /etc/systemd/system/defundd.service
sudo systemctl daemon-reload
sudo rm -f $(which defundd) 
sudo rm -rf $HOME/.defund
sudo rm -rf $HOME/defund
```
