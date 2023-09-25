# Official Links
[Website](https://www.mande.network/) [Discord](https://discord.gg/Q43H94fG7X) [Twitter](https://twitter.com/MandeNetwork) [Official Document](https://github.com/mande-labs/testnet-1)

# Explorer
[Explorer](https://explorer.stavr.tech/mande-chain/staking)


# Install Node Guide Mande
### Setting up variables
Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
MANDE_PORT=40
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export MANDE_CHAIN_ID=mande-testnet-2" >> $HOME/.bash_profile
echo "export MANDE_PORT=${MANDE_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Update Packages and Depencies
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Depencies
```bash
sudo apt install curl build-essential git wget jq lz4 make gcc tmux chrony -y
```

### Install GO
```bash
if ! [ -x "$(command -v go)" ]; then
  ver="1.19.6"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

### Download binaries
```bash
cd $HOME
git clone https://github.com/mande-labs/mande-chain.git
cd mande-chain
git checkout v1.2.2
ignite chain build --release
```

### Install Wasm Library
```bash
cd $HOME
go install  github.com/CosmWasm/wasmvm@v1.0.0
cd $HOME/go/pkg/mod/github.com/!cosm!wasm/wasmvm@v1.0.0/api
chmod +x libwasmvm.x86_64.so 
cp -r $HOME/go/pkg/mod/github.com/!cosm!wasm/wasmvm@v1.0.0/api/libwasmvm.x86_64.so /usr/lib/
```

### Config app
```bash
mande-chaind config chain-id $MANDE_CHAIN_ID
mande-chaind config keyring-backend test
mande-chaind config node tcp://localhost:${MANDE_PORT}657
```

## Init app
```bash
mande-chaind init $NODENAME --chain-id $MANDE_CHAIN_ID
```

### Download genesis and addrbook
```bash
wget -O $HOME/.mande-chain/config/genesis.json "https://raw.githubusercontent.com/mande-labs/testnet-2/main/genesis.json"
```

## Set seeds and peers
```bash
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.mande-chain/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.mande-chain/config/config.toml
peers="dbd1f5b01f010b9e6ae6d9f293d2743b03482db5@34.171.132.212:26656,1d1da5742bdd281f0829124ec60033f374e9ddac@34.170.16.69:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.mande-chain/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.mande-chain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.mande-chain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.mande-chain/config/config.toml
```

### Set minimum gas price
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005mand\"/;" ~/.mande-chain/config/app.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${MANDE_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${MANDE_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${MANDE_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${MANDE_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${MANDE_PORT}660\"%" $HOME/.mande-chain/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${MANDE_PORT}317\"%; s%^address = \":8080\"%address = \":${MANDE_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${MANDE_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${MANDE_PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${MANDE_PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${MANDE_PORT}546\"%" $HOME/.mande-chain/config/app.toml
```

### Config pruning (Optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.mande-chain/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.mande-chain/config/config.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.mande-chain/config/config.toml
```

### Update ~/.mande-chain/config/config.toml
```bash
max_packet_msg_payload_size = 10240
```

### Reset chain data
```bash
mande-chaind tendermint unsafe-reset-all --home $HOME/.mande-chain
```

### Create service
```bash
sudo tee /etc/systemd/system/mande-chaind.service > /dev/null <<EOF
[Unit]
Description=Mande
After=network-online.target

[Service]
User=$USER
ExecStart=$(which mande-chaind) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Register and start service
```bash
sudo systemctl daemon-reload
sudo systemctl enable mande-chaind
sudo systemctl restart mande-chaind && sudo journalctl -fu mande-chaind -o cat
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics
```bash
mande-chaind keys add $WALLET
```

To recover existing keys use
```bash
mande-chaind keys add $WALLET --recover
```

List of wallets
```bash
mande-chaind keys list
```

### Save wallet info
Add wallet and valoper address into variables
```bash
MANDE_WALLET_ADDRESS=$(mande-chaind keys show $WALLET -a)
MANDE_VALOPER_ADDRESS=$(mande-chaind keys show $WALLET --bech val -a)
echo 'export MANDE_WALLET_ADDRESS='${MANDE_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export MANDE_VALOPER_ADDRESS='${MANDE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### To check your wallet balance:
```bash
mande-chaind query bank balances $MANDE_WALLET_ADDRESS
```

### Create validator
After your node is synced, create validator
```bash
mande-chaind tx staking create-validator \
  --amount 0cred \
  --from $WALLET \
  --pubkey  $(mande-chaind tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $MANDE_CHAIN_ID \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --fees=1000mand \
  --gas-adjustment=1.15 -y
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu mande-chaind -o cat
```

Start service
```bash
sudo systemctl start mande-chaind
```

Stop service
```bash
sudo systemctl stop mande-chaind
```

Restart service
```bash
sudo systemctl restart mande-chaind
```

### Node info
Synchronization info
```bash
mande-chaind status 2>&1 | jq .SyncInfo
```

Validator info
```bash
mande-chaind status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
mande-chaind status 2>&1 | jq .NodeInfo
```

Show node id
```bash
mande-chaind tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
mande-chaind keys list
```

Recover wallet
```bash
mande-chaind keys add $WALLET --recover
```

Delete wallet
```bash
mande-chaind keys delete $WALLET
```

Get wallet balance
```bash
mande-chaind query bank balances $MANDE_WALLET_ADDRESS
```

Transfer funds
```bash
mande-chaind tx bank send $MANDE_WALLET_ADDRESS <TO_NIBIRU_WALLET_ADDRESS> 10000000mand --fees=1000mand
```

### Voting
```bash
mande-chaind tx gov vote 1 yes --from $WALLET --chain-id=$MANDE_CHAIN_ID --fees=1000mand
```

### Staking, Delegation and Rewards
Delegate stake
```bash
mande-chaind tx staking delegate $MANDE_VALOPER_ADDRESS 10000000mand --from=$WALLET --chain-id=$MANDE_CHAIN_ID --fees=1000mand
```

Redelegate stake from validator to another validator
```bash
mande-chaind tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000mand --from=$WALLET --chain-id=$MANDE_CHAIN_ID --fees=1000mand
```

Withdraw all rewards
```bash
mande-chaind tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$MANDE_CHAIN_ID --fees=1000mand
```

Withdraw rewards with commision
```bash
mande-chaind tx distribution withdraw-rewards $MANDE_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$MANDE_CHAIN_ID --fees=1000mand
```

### Validator management
Edit validator
```bash
mande-chaind tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$MANDE_CHAIN_ID \
  --from=$WALLET \
  --fees=1000mand -y
```

Unjail validator
```bash
mande-chaind tx slashing unjail \
  --from=$WALLET \
  --chain-id=$MANDE_CHAIN_ID \
  --fees=1000mand -y
```

### Delete node
```bash
sudo systemctl stop mande-chaind
sudo systemctl disable mande-chaind
sudo rm /etc/systemd/system/mande* -rf
sudo rm $(which mande-chaind) -rf
sudo rm $HOME/.mande-chain* -rf
sudo rm $HOME/mande-chain -rf
sed -i '/MANDE_/d' ~/.bash_profile
```
