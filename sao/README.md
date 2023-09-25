# Install Guide SAO Beta

## [Website](https://www.sao.network/) | [Discord](https://discord.gg/f4xzfvPhhA) | [Twitter](https://twitter.com/SAONetwork) | :satellite:[EXPLORER](https://explorer.moonbridge.team/sao-beta)

**Chain ID:** sao-20230629 | **Latest Version:** v0.1.8 | **Custom Port:** 134

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
SAO_PORT=134
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
git clone https://github.com/SaoNetwork/sao-consensus.git
cd sao-consensus
git checkout v0.1.8
make install
saod version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
saod config node tcp://localhost:${SAO_PORT}57
saod config chain-id sao-20230629
saod config keyring-backend test
saod init $MONIKER --chain-id sao-20230629

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/SAO/genesis.json > $HOME/.sao/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/SAO/addrbook.json > $HOME/.sao/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.sao/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0usct\"|" $HOME/.sao/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.sao/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.sao/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.sao/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.sao/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.sao/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.sao/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${SAO_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${SAO_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${SAO_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${SAO_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${SAO_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${SAO_PORT}66\"%" $HOME/.sao/config/config.toml
sed -i -e "s%:1317%:${SAO_PORT}17%g; s%:8080%:${SAO_PORT}80%g; s%:9090%:${SAO_PORT}90%g; s%:9091%:${SAO_PORT}91%g; s%:8545%:${SAO_PORT}45%g; s%:8546%:${SAO_PORT}46%g; s%:6065%:${SAO_PORT}65%g" $HOME/.sao/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/saod.service > /dev/null <<EOF
[Unit]
Description=SAO Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which saod) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable saod
```

## Start service and check the logs

```bash
sudo systemctl start saod && sudo journalctl -u saod -f --no-hostname -o cat
```

## Create wallet

```bash
saod keys add wallet
```

## Check wallet balance

```bash
saod q bank balances $(saod keys show wallet -a)
```

## Create validator

```bash
saod tx staking create-validator \
  --amount 1000000usct \
  --pubkey $(saod tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id sao-20230629 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.5 \
  --gas auto \
  --gas-prices 0usct \
  -y
```

## Firewall security

```bash
sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${SAO_PORT}56/tcp
sudo ufw enable
```

## Delete node

```bash
sudo systemctl stop saod
sudo systemctl disable saod
sudo rm -rf /etc/systemd/system/saod.service
sudo systemctl daemon-reload
sudo rm -f $(which saod) 
sudo rm -rf $HOME/.sao 
sudo rm -rf $HOME/sao-consensus
```
