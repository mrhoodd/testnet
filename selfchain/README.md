# Install Guide Self Chain

## [Website](https://selfchain.xyz/) | [Twitter](https://twitter.com/selfchainxyz) | [Discord](https://discord.gg/selfchainxyz) | :satellite:[Explorer](https://explorer.moonbridge.team/nibiru-test)

**Chain ID:** self-dev-1 | **Latest Version:**  | **Custom Port:** 141

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
SELF_PORT=141
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
wget https://ss-t.self.nodestake.top/selfchaind
chmod +x selfchaind
mv selfchaind /root/go/bin/
selfchaind version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
selfchaind config node tcp://localhost:${SELF_PORT}57
selfchaind config chain-id self-dev-1
selfchaind config keyring-backend test
selfchaind init $MONIKER --chain-id self-dev-1

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/genesis.json > $HOME/.selfchain/config/genesis.json
curl -Ls https://raw.githubusercontent.com/MrHoodd/MainnetNodes/main/Nois/addrbook.json > $HOME/.selfchain/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.selfchain/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0uself\"|" $HOME/.selfchain/config/app.toml

# Setting pruning
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.selfchain/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.selfchain/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.selfchain/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.selfchain/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.selfchain/config/config.toml

# Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.selfchain/config/config.toml

# Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${SELF_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${SELF_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${SELF_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${SELF_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${SELF_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${SELF_PORT}66\"%" $HOME/.selfchain/config/config.toml
sed -i -e "s%:1317%:${SELF_PORT}17%g; s%:8080%:${SELF_PORT}80%g; s%:9090%:${SELF_PORT}90%g; s%:9091%:${SELF_PORT}91%g; s%:8545%:${SELF_PORT}45%g; s%:8546%:${SELF_PORT}46%g; s%:6065%:${SELF_PORT}65%g" $HOME/.selfchain/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/selfchaind.service > /dev/null <<EOF
[Unit]
Description=SelfChain Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which selfchaind) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable selfchaind
```

## Start service and check the logs

```bash
sudo systemctl start selfchaind && sudo journalctl -u selfchaind -f --no-hostname -o cat
```

## Create wallet

```bash
selfchaind keys add wallet
```

## Check wallet balance

```bash
selfchaind q bank balances $(selfchaind keys show wallet -a)
```

## Create validator

```bash
selfchaind tx staking create-validator \
  --amount 1000000uself \
  --pubkey $(selfchaind tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id self-dev-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.0uself \
  -y
```

## Delete node

```bash
sudo systemctl stop selfchaind
sudo systemctl disable selfchaind
sudo rm -rf /etc/systemd/system/selfchaind.service
sudo systemctl daemon-reload
sudo rm -f $(which selfchaind) 
sudo rm -rf $HOME/.selfchain
```
