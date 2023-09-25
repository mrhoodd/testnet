# Install Guide Entangle

## [Website](https://entangle.fi/) | [Twitter](https://twitter.com/Entanglefi) | [Discord](https://discord.com/invite/entanglefi) | :satellite:[Explorer](https://explorer.moonbridge.team/nibiru-test)

**Chain ID:** entangle_33133-1 | **Latest Version:** v1.0.1 | **Custom Port:** 142

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
ENTANGLE_PORT=142
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
git clone https://github.com/Entangle-Protocol/entangle-blockchain.git
cd entangle-blockchain
make install
entangled version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
entangled config node tcp://localhost:${ENTANGLE_PORT}57
entangled config chain-id entangle_33133-1
entangled config keyring-backend test
entangled init $MONIKER --chain-id entangle_33133-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/genesis.json > $HOME/.entangled/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/addrbook.json > $HOME/.entangled/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.entangled/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"10aNGL\"|" $HOME/.entangled/config/app.toml

# Setting pruning
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.entangled/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.entangled/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.entangled/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.entangled/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.entangled/config/config.toml

# Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.entangled/config/config.toml

# Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ENTANGLE_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${ENTANGLE_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ENTANGLE_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ENTANGLE_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ENTANGLE_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ENTANGLE_PORT}66\"%" $HOME/.entangled/config/config.toml
sed -i -e "s%:1317%:${ENTANGLE_PORT}17%g; s%:8080%:${ENTANGLE_PORT}80%g; s%:9090%:${ENTANGLE_PORT}90%g; s%:9091%:${ENTANGLE_PORT}91%g; s%:8545%:${ENTANGLE_PORT}45%g; s%:8546%:${ENTANGLE_PORT}46%g; s%:6065%:${ENTANGLE_PORT}65%g" $HOME/.entangled/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/entangled.service > /dev/null <<EOF
[Unit]
Description=Entangle Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which entangled) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable entangled
```

## Start service and check the logs

```bash
sudo systemctl start entangled && sudo journalctl -u entangled -f --no-hostname -o cat
```

## Create wallet

```bash
entangled keys add wallet
```

## Check wallet balance

```bash
entangled q bank balances $(entangled keys show wallet -a)
```

## Create validator

```bash
entangled tx staking create-validator \
  --amount 5000000000000000000aNGL \
  --pubkey $(entangled tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id entangle_33133-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 10aNGL \
  -y
```

## Delete node

```bash
sudo systemctl stop entangled
sudo systemctl disable entangled
sudo rm -rf /etc/systemd/system/entangled.service
sudo systemctl daemon-reload
sudo rm -f $(which entangled) 
sudo rm -rf $HOME/.entangled
sudo rm -rf $HOME/entangle-blockchain
```
