# Install Guide Mantra

## [Website](https://www.mantrachain.io/) | [Twitter](https://twitter.com/MANTRA_Chain) | [Discord](https://discord.gg/gfks4TwAJV) | :satellite:[Explorer](https://explorer.moonbridge.team/mantra-test)

## Public endpoints
- API: https://mantra-test.api.moonbridge.team
- RPC: https://mantra-test.rpc.moonbridge.team

**Chain ID:** mantrachain-1 | **Latest Version:**  | **Custom Port:** 143

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
MANTRA_PORT=143
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
# Install CosmWasm Library
sudo wget -O /usr/lib/libwasmvm.x86_64.so https://github.com/CosmWasm/wasmvm/releases/download/v1.3.0/libwasmvm.x86_64.so
wget https://github.com/MANTRA-Finance/public/raw/main/mantrachain-testnet/mantrachaind-linux-amd64.zip
unzip mantrachaind-linux-amd64.zip
mv mantrachaind $HOME/go/bin
rm mantrachaind-linux-amd64.zip
mantrachaind version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
mantrachaind config node tcp://localhost:${MANTRA_PORT}57
mantrachaind config chain-id mantrachain-1
mantrachaind config keyring-backend test
mantrachaind init $MONIKER --chain-id mantrachain-1

# Download genesis and addrbook
curl -Ls https://moonbridge.team/snapshots/testnet/mantra/genesis.json > $HOME/.mantrachain/config/genesis.json
curl -Ls https://moonbridge.team/snapshots/testnet/mantra/addrbook.json > $HOME/.mantrachain/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.mantrachain/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0uaum\"|" $HOME/.mantrachain/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.mantrachain/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.mantrachain/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.mantrachain/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.mantrachain/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.mantrachain/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.mantrachain/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${MANTRA_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${MANTRA_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${MANTRA_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${MANTRA_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${MANTRA_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${MANTRA_PORT}66\"%" $HOME/.mantrachain/config/config.toml
sed -i -e "s%:1317%:${MANTRA_PORT}17%g; s%:8080%:${MANTRA_PORT}80%g; s%:9090%:${MANTRA_PORT}90%g; s%:9091%:${MANTRA_PORT}91%g; s%:8545%:${MANTRA_PORT}45%g; s%:8546%:${MANTRA_PORT}46%g; s%:6065%:${MANTRA_PORT}65%g" $HOME/.mantrachain/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/mantrachaind.service > /dev/null <<EOF
[Unit]
Description=Mantra Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.mantrachain
ExecStart=$(which mantrachaind) start --home $HOME/.mantrachain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable mantrachaind
```

## Start service and check the logs

```bash
sudo systemctl start mantrachaind && sudo journalctl -u mantrachaind -f --no-hostname -o cat
```

## Create wallet

```bash
mantrachaind keys add wallet
```

## Recover wallet

```bash
mantrachaind keys add wallet --recover
```

## Check wallet balance

```bash
mantrachaind q bank balances $(mantrachaind keys show wallet -a)
```

## Create validator

```bash
mantrachaind tx staking create-validator \
  --amount 1000000uaum \
  --pubkey $(mantrachaind tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id mantrachain-1 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.0uaum \
  -y
```

## Delete node

```bash
sudo systemctl stop mantrachaind
sudo systemctl disable mantrachaind
sudo rm -rf /etc/systemd/system/mantrachaind.service
sudo systemctl daemon-reload
sudo rm -f $(which mantrachaind) 
sudo rm -rf $HOME/.mantrachain
```
