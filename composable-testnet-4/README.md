
# Install Guide Composable Testnet-4

## [Website](https://www.composable.finance/) | [Discord](https://discord.gg/composable) | [Twitter](https://twitter.com/ComposableFin) | :satellite:[Explorer](https://explorer.moonbridge.team/composable-test4)

## Public endpoints

- API: https://composable-test4.api.moonbridge.team
- RPC: https://composable-test4.rpc.moonbridge.team

**Chain ID:** banksy-testnet-4 | **Latest Version:** v5.2.4-testnet4 | **Custom Port:** 145

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
BANKSY_PORT=145
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
git clone https://github.com/notional-labs/composable-centauri
cd composable-centauri
git checkout v5.2.4-testnet4
make install
centaurid version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
centaurid config node tcp://localhost:${BANKSY_PORT}57
centaurid config chain-id banksy-testnet-4
centaurid config keyring-backend test
centaurid init $MONIKER --chain-id banksy-testnet-4

# Download genesis and addrbook
curl -Ls https://raw.githubusercontent.com/notional-labs/composable-networks/main/banksy-testnet-4/genesis.json > $HOME/.banksy/config/genesis.json
curl -Ls https://moonbridge.team/snapshots/testnet/composabletest4/addrbook.json > $HOME/.banksy/config/addrbook.json

# Set seeds and peers
SEEDS="a89d3d9fc0465615aa1100dcf53172814aa2b8cf@168.119.91.22:2260"
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.banksy/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ppica\"|" $HOME/.banksy/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.banksy/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.banksy/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.banksy/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.banksy/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.banksy/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.banksy/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${BANKSY_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${BANKSY_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${BANKSY_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${BANKSY_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${BANKSY_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${BANKSY_PORT}66\"%" $HOME/.banksy/config/config.toml
sed -i -e "s%:1317%:${BANKSY_PORT}17%g; s%:8080%:${BANKSY_PORT}80%g; s%:9090%:${BANKSY_PORT}90%g; s%:9091%:${BANKSY_PORT}91%g; s%:8545%:${BANKSY_PORT}45%g; s%:8546%:${BANKSY_PORT}46%g; s%:6065%:${BANKSY_PORT}65%g" $HOME/.banksy/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/centaurid.service > /dev/null <<EOF
[Unit]
Description=Composable Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.banksy
ExecStart=$(which centaurid) start --home $HOME/.banksy
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable centaurid
```

## Start service and check the logs

```bash
sudo systemctl start centaurid && sudo journalctl -u centaurid -f --no-hostname -o cat
```

## Create wallet

```bash
centaurid keys add wallet
```

## Recover wallet

```bash
centaurid keys add wallet --recover
```

## Check wallet balance

```bash
centaurid q bank balances $(centaurid keys show wallet -a)
```

## Create validator

```bash
centaurid tx staking create-validator \
  --amount 1000000000000ppica \
  --pubkey $(centaurid tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id banksy-testnet-4 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0ppica \
  -y
```

## Delete node

```bash
sudo systemctl stop centaurid
sudo systemctl disable centaurid
sudo rm -rf /etc/systemd/system/centaurid.service
sudo systemctl daemon-reload
sudo rm -f $(which centaurid) 
sudo rm -rf $HOME/.banksy
sudo rm -rf $HOME/composable-centauri
```
