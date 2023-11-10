```bash
cd $HOME/composable-centauri
git pull
git checkout v6.2.3-testnet
make install
layerd version --long | grep -e commit -e version
sudo systemctl restart layerd && sudo journalctl -u layerd -f --no-hostname -o cat
```
