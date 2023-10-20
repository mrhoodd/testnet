
## Key management

Add new key

```bash
althea keys add wallet
```

Recover existing key

```bash
althea keys add wallet --recover
```

List all keys

```bash
althea keys list
```

Delete key

```bash
althea keys delete wallet
```

Export key to the file

```bash
althea keys export wallet
```

Import key from the file

```bash
althea keys import wallet wallet.backup
```

Wallet balance

```bash
althea q bank balances $(althea keys show wallet -a)
```

## Token management

Withdraw rewards from all validators

```bash
althea tx distribution withdraw-all-rewards --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

Withdraw rewards and commissions from your validator

```bash
althea tx distribution withdraw-rewards $(althea keys show wallet --bech val -a) --commission --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

Delegate tokens to yourself

```bash
althea tx staking delegate $(althea keys show wallet --bech val -a) 1000000aalthea --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

Delegate tokens to validator

```bash
althea tx staking delegate <TO_VALOPER_ADDRESS> 1000000aalthea --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

Redelegate tokens to another validator

```bash
althea tx staking redelegate $(althea keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 1000000aalthea --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

Unbond tokens from your validator

```bash
althea tx staking unbond $(althea keys show wallet --bech val -a) 1000000aalthea --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

Send tokens to the wallet

```bash
althea tx bank send wallet <TO_WALLET_ADDRESS> 1000000aalthea --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

## Validator management

Validator info

```bash
althea status 2>&1 | jq .ValidatorInfo
```

Validator details

```bash
althea q staking validator $(althea keys show wallet --bech val -a)
```

Check if validator key is correct

```bash
[[ $(althea q staking validator $(althea keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(althea status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

List all active validators

```bash
althea q staking validators -oj --limit=1000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

List all inactive validators

```bash
althea q staking validators -oj --limit=1000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

Edit existing validator

```bash
althea tx staking edit-validator \
  --new-moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id althea_417834-3 \
  --commission-rate 0.10 \
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.025aalthea \
  -y
```

Jail reason

```bash
althea query slashing signing-info $(althea tendermint show-validator)
```

Unjail validator

```bash
althea tx slashing unjail --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

## Governance

Create a new offer

```bash
althea tx gov submit-proposal \
  --title "" \
  --description "" \
  --deposit 1000000000000000000aalthea \
  --type Text
  --from wallet \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.025aalthea \
  -y 
```

List all proposals

```bash
althea query gov proposals
```

View proposal by ID

```bash
althea query gov proposal 1
```

Vote "YES"

```bash
althea tx gov vote 1 yes --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

Vote "NO"

```bash
althea tx gov vote 1 no --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

Vote "ABSTAIN"

```bash
althea tx gov vote 1 abstain --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

Vote "NOWITHVETO"

```bash
althea tx gov vote 1 NoWithVeto --from wallet --chain-id althea_417834-3 --gas-adjustment 1.4 --gas auto --gas-prices 0.025aalthea -y
```

## Maintenance

Get sync info

```bash
althea status 2>&1 | jq .SyncInfo
```

Get node peer

```bash
echo $(althea tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.althea/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

Get live peers

```bash
curl -sS http://localhost:12357/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Enable Prometheus

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.althea/config/config.toml
```

Set minimum gas price

```bash
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025aalthea\"|" $HOME/.althea/config/app.toml
```

Disable indexer

```bash
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.althea/config/config.toml
```

Enable indexer

```bash
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.althea/config/config.toml
```

Update pruning

```bash
sed -i 's|^pruning *=.*|pruning = "custom"|' $HOME/.althea/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.althea/config/app.toml
sed -i 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.althea/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.althea/config/app.toml
```

Update ports

```bash
CUSTOM_PORT=123
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.althea/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.althea/config/app.toml
```

Reset chain data

```bash
althea tendermint unsafe-reset-all --keep-addr-book --home $HOME/.althea
```

Delete node

```bash
sudo systemctl stop althea
sudo systemctl disable althea
sudo rm -rf /etc/systemd/system/althea.service
sudo systemctl daemon-reload
sudo rm -f $(which althea) 
sudo rm -rf $HOME/.althea
sudo rm -rf $HOME/althea-chain
```

## Service Management

Status service

```bash
sudo systemctl status althea
```

Start service

```bash
sudo systemctl start althea
```

Stop service

```bash
sudo systemctl stop althea
```

Restart service

```bash
sudo systemctl restart althea
```

Logs service

```bash
sudo journalctl -u althea -f --no-hostname -o cat
```

Reload service

```bash
sudo systemctl daemon-reload
```

Enable service

```bash
sudo systemctl enable althea
```

Disable service

```bash
sudo systemctl disable althea
```
