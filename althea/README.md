# Install Guide Althea L1 Testnet 4

## [Website](http://althea.net) | [Twitter](https://twitter.com/AltheaNetwork) | [Discord](https://t.co/foHCl8hl6G) | :satellite:[Explorer](https://explorer.moonbridge.team/althea-test-4)

**Chain ID:** althea_417834-3 | **Latest Version:** v0.5.5 | **Custom Port:** 123

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
ALTHEA_PORT=123
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
wget https://github.com/althea-net/althea-L1/releases/download/v0.5.5/althea-linux-amd64
chmod +x althea-linux-amd64
sudo mv althea-linux-amd64 $HOME/go/bin/althea
althea version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
althea config node tcp://localhost:${ALTHEA_PORT}57
althea config chain-id althea_417834-3
althea config keyring-backend test
althea init $MONIKER --chain-id althea_417834-3

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Althea/althea-testnet-4/genesis.json > $HOME/.althea/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Althea/althea-testnet-4/addrbook.json > $HOME/.althea/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS="bc47f3e8f9134a812462e793d8767ef7334c0119@chainripper-2.althea.net:23296"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.althea/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025aalthea\"|" $HOME/.althea/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.althea/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.althea/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.althea/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.althea/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.althea/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.althea/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ALTHEA_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${ALTHEA_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ALTHEA_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ALTHEA_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ALTHEA_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ALTHEA_PORT}66\"%" $HOME/.althea/config/config.toml
sed -i -e "s%:1317%:${ALTHEA_PORT}17%g; s%:8080%:${ALTHEA_PORT}80%g; s%:9090%:${ALTHEA_PORT}90%g; s%:9091%:${ALTHEA_PORT}91%g; s%:8545%:${ALTHEA_PORT}45%g; s%:8546%:${ALTHEA_PORT}46%g; s%:6065%:${ALTHEA_PORT}65%g" $HOME/.althea/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/althea.service > /dev/null <<EOF
[Unit]
Description=Althea Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which althea) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable althea
```

## Start service and check the logs

```bash
sudo systemctl start althea && sudo journalctl -u althea -f --no-hostname -o cat
```

## Create wallet

```bash
althea keys add wallet
```

## Check wallet balance

```bash
althea q bank balances $(althea keys show wallet -a)
```

## Create validator

```bash
althea tx staking create-validator \
  --amount 999000000000000000000aalthea \
  --pubkey $(althea tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id althea_417834-3 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.025aalthea \
  -y
```

## Firewall security

```bash
sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${ALTHEA_PORT}56/tcp
sudo ufw enable
```

## Delete node

```bash
sudo systemctl stop althea
sudo systemctl disable althea
sudo rm -rf /etc/systemd/system/althea.service
sudo systemctl daemon-reload
sudo rm -f $(which althea) 
sudo rm -rf $HOME/.althea
sudo rm -rf $HOME/althea-chain
```
