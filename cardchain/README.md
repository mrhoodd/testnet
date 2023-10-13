# Install Guide Cardchain

## [Website](https://crowdcontrol.network/#/) | [Twitter](https://twitter.com/CrowdControlNet?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E1577353712963665922%7Ctwgr%5Eeffb4472653f0071a6ba1c0a52b2f82372d78fc6%7Ctwcon%5Es1_&ref_url=https%3A%2F%2Flinktr.ee%2Fcrowdcontrolnet) | [Discord](https://discord.gg/yPA3aKe) | :satellite:[Explorer](https://explorer.moonbridge.team/cardchain-test)

## Public endpoints

- API: https://cardchain-test.api.moonbridge.team
- RPC: https://cardchain-test.rpc.moonbridge.team

**Chain ID:** cardtestnet-4 | **Latest Version:** v0.9.1 | **Custom Port:** 124

:red_circle:Specify the name of your moniker (validator) which will be visible in the explorer

```bash
MONIKER="YOUR_MONIKER_NAME"
```

Custom port

```bash
CARDCHAIN_PORT=124
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
git clone https://github.com/DecentralCardGame/Cardchain
wget https://github.com/DecentralCardGame/Cardchain/releases/download/v0.9.1/Cardchaind
chmod +x Cardchaind
mv $HOME/Cardchaind $HOME/go/bin
Cardchaind version --long | grep -e commit -e version
```

## Config and Init node

```bash
# Set node configuration
Cardchaind config node tcp://localhost:${CARDCHAIN_PORT}57
Cardchaind config chain-id cardtestnet-4
Cardchaind config keyring-backend test
Cardchaind init $MONIKER --chain-id cardtestnet-4

# Download genesis and addrbook
curl -Ls https://moonbridge.team/snapshots/testnet/cardchain/genesis.json > $HOME/.Cardchain/config/genesis.json
curl -Ls https://moonbridge.team/snapshots/testnet/cardchain/addrbook.json > $HOME/.Cardchain/config/addrbook.json

# Set seeds and peers
SEEDS=""
PEERS="1ed98c796bcdd0faf5a7ad8793d229e3c7d89543@lxgr.xyz:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.Cardchain/config/config.toml

# Setting minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0ubpf\"|" $HOME/.Cardchain/config/app.toml

# Setting pruning
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.Cardchain/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.Cardchain/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.Cardchain/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.Cardchain/config/app.toml

# Disable indexer (optional)
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.Cardchain/config/config.toml

## Enable Prometheus (optional)
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.Cardchain/config/config.toml

## Setting custom ports
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CARDCHAIN_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${CARDCHAIN_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CARDCHAIN_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CARDCHAIN_PORT}56\"%; s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${CARDCHAIN_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CARDCHAIN_PORT}66\"%" $HOME/.Cardchain/config/config.toml
sed -i -e "s%:1317%:${CARDCHAIN_PORT}17%g; s%:8080%:${CARDCHAIN_PORT}80%g; s%:9090%:${CARDCHAIN_PORT}90%g; s%:9091%:${CARDCHAIN_PORT}91%g; s%:8545%:${CARDCHAIN_PORT}45%g; s%:8546%:${CARDCHAIN_PORT}46%g; s%:6065%:${CARDCHAIN_PORT}65%g" $HOME/.Cardchain/config/app.toml
```

## Create service

```bash
sudo tee /etc/systemd/system/Cardchaind.service > /dev/null <<EOF
[Unit]
Description=Cardchain Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which Cardchaind) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable Cardchaind
```

## Start service and check the logs

```bash
sudo systemctl start Cardchaind && sudo journalctl -u Cardchaind -f --no-hostname -o cat
```

## Create wallet

```bash
Cardchaind keys add wallet
```

## Check wallet balance

```bash
Cardchaind q bank balances $(Cardchaind keys show wallet -a)
```

## Create validator

```bash
Cardchaind tx staking create-validator \
  --amount 1000000ubpf \
  --pubkey $(Cardchaind tendermint show-validator) \
  --moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id cardtestnet-4 \
  --commission-rate 0.10 \
  --commission-max-rate 0.20 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0ubpf \
  -y
```

## Delete node

```bash
sudo systemctl stop Cardchaind
sudo systemctl disable Cardchaind
sudo rm -rf /etc/systemd/system/Cardchaind.service
sudo systemctl daemon-reload
sudo rm -f $(which Cardchaind) 
sudo rm -rf $HOME/.Cardchain
sudo rm -rf $HOME/Cardchain
```
