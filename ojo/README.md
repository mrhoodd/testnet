# Install Guide Ojo

## [Website](https://ojo.network) | [Discord](https://discord.gg/fd8Yrex8nC) | [Twitter](https://twitter.com/ojo_network) | :satellite:[Explorer](https://ojo.explorers.guru/validators)

**Chain ID:** ojo-devnet | **Latest Version:** v0.1.2 | **Custom Port:** 135

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
OJO_PORT=135
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
git clone https://github.com/ojo-network/ojo.git
cd ojo
git checkout v0.1.2
make install
ojod version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
ojod config node tcp://localhost:${OJO_PORT}57
ojod config chain-id ojo-devnet
ojod config keyring-backend test
ojod init $MONIKER --chain-id ojo-devnet

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/genesis.json > $HOME/.ojo/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/addrbook.json > $HOME/.ojo/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.ojo/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uojo\"|" $HOME/.ojo/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.ojo/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.ojo/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.ojo/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.ojo/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.ojo/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.ojo/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${OJO_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${OJO_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OJO_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${OJO_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${OJO_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${OJO_PORT}66\"%" $HOME/.ojo/config/config.toml
sed -i -e "s%:1317%:${OJO_PORT}17%g; s%:8080%:${OJO_PORT}80%g; s%:9090%:${OJO_PORT}90%g; s%:9091%:${OJO_PORT}91%g; s%:8545%:${OJO_PORT}45%g; s%:8546%:${OJO_PORT}46%g; s%:6065%:${OJO_PORT}65%g" $HOME/.ojo/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/ojod.service > /dev/null <<EOF
[Unit]
Description=Ojo Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which ojod) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable ojod
```

## Start service and check the logs

```bash
sudo systemctl start ojod && sudo journalctl -u ojod -f --no-hostname -o cat
```

## Create wallet

```bash
ojod keys add wallet
```

## Check wallet balance

```bash
ojod q bank balances $(ojod keys show wallet -a)
```

## Create validator

```bash
ojod tx staking create-validator \
  --amount 1000000uojo \
  --pubkey $(ojod tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id ojo-devnet \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0uojo \
  -y
```

## Delete node

```bash
sudo systemctl stop ojod
sudo systemctl disable ojod
sudo rm -rf /etc/systemd/system/ojod.service
sudo systemctl daemon-reload
sudo rm -f $(which ojod) 
sudo rm -rf $HOME/.ojo
sudo rm -rf $HOME/ojo
```
