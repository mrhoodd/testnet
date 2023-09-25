# Install Guide BonusBlock

## [Website](https://bonusblock.io/) | [Telegram](https://t.me/bonusblock) | [Twitter](https://twitter.com/bonus_block) | :satellite:[Explorer](https://explorer.moonbridge.team/bonusblock-test)

**Chain ID:** blocktopia-01 | **Latest Version:**  | **Custom Port:** 130

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
BONUS_PORT=130
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
git clone https://github.com/BBlockLabs/BonusBlock-chain
cd BonusBlock-chain
make install
bonus-blockd version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
bonus-blockd config node tcp://localhost:${BONUS_PORT}57
bonus-blockd config chain-id blocktopia-01
bonus-blockd config keyring-backend test
bonus-blockd init $MONIKER --chain-id blocktopia-01

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/genesis.json > $HOME/.bonusblock/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/TestnetNodes/main/Empower/addrbook.json > $HOME/.bonusblock/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.bonusblock/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025ubonus\"|" $HOME/.bonusblock/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.bonusblock/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.bonusblock/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.bonusblock/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.bonusblock/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.bonusblock/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.bonusblock/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${BONUS_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${BONUS_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${BONUS_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${BONUS_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${BONUS_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${BONUS_PORT}66\"%" $HOME/.bonusblock/config/config.toml
sed -i -e "s%:1317%:${BONUS_PORT}17%g; s%:8080%:${BONUS_PORT}80%g; s%:9090%:${BONUS_PORT}90%g; s%:9091%:${BONUS_PORT}91%g; s%:8545%:${BONUS_PORT}45%g; s%:8546%:${BONUS_PORT}46%g; s%:6065%:${BONUS_PORT}65%g" $HOME/.bonusblock/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/bonus-blockd.service > /dev/null <<EOF
[Unit]
Description=BonusBlock Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which bonus-blockd) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable bonus-blockd
```

## Start service and check the logs

```bash
sudo systemctl start bonus-blockd && sudo journalctl -u bonus-blockd -f --no-hostname -o cat
```

## Create wallet

```bash
bonus-blockd keys add wallet
```

## Check wallet balance

```bash
bonus-blockd q bank balances $(bonus-blockd keys show wallet -a)
```

## Create validator

```bash
bonus-blockd tx staking create-validator \
  --amount 1000000ubonus \
  --pubkey $(bonus-blockd tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id blocktopia-01 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.025ubonus \
  -y
```

## Delete node

```bash
sudo systemctl stop bonus-blockd
sudo systemctl disable bonus-blockd
sudo rm -rf /etc/systemd/system/bonus-blockd.service
sudo systemctl daemon-reload
sudo rm -f $(which bonus-blockd) 
sudo rm -rf $HOME/.bonusblock
sudo rm -rf $HOME/BonusBlock-chain
```
