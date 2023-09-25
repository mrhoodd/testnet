# Install Guide Bundlr

## [Website](https://bundlr.network) | [Twitter](https://twitter.com/BundlrNetwork) | [Discord](https://discord.gg/JpwRkXFZ)

## Update Packages

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

## Install Depencies

```bash
sudo apt-get install curl wget git jq libpq-dev libssl-dev build-essential pkg-config openssl ocl-icd-opencl-dev libopencl-clang-dev libgomp1 -y
```

## Install 'docker' and 'docker compose'

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## Install 'rust'

```bash
$ curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
```

## Check version

```bash
rustc --version
cargo --version
```

## Install 'nodejs' and 'npm'

```bash
curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash - && \
sudo apt-get install nodejs -y && \
echo -e "\nnodejs > $(node --version).\nnpm  >>> v$(npm --version).\n"
```

## Create directory 'bundlr'

```bash
mkdir $HOME/bundlr; cd $HOME/bundlr
```

## Clone repository 'validator-rust'

```bash
git clone \
--recurse-submodules https://github.com/Bundlr-Network/validator-rust.git
```

## Generating 'wallet.json'

```bash
cd $HOME/bundlr/validator-rust && \
cargo run --bin wallet-tool create > wallet.json
```

## Back up the file 'wallet.json'

```bash
$HOME/bundlr/validator-rust/wallet.json
```

## View address

```bash
cd $HOME/bundlr/validator-rust && \
cargo run --bin wallet-tool show-address \
--wallet wallet.json | jq ".address" | tr -d '"'
```

## Go to the page with the [faucet](https://bundlr.network/faucet) and request tokens

## Save and import variables into system

```bash
PORT=(Specify your meaning)
ADDRESS=(Specify your meaning)
echo "export BUNDLR_PORT="${PORT}"" >> $HOME/.bash_profile
echo "export BUNDLR_ADDRESS="${ADDRESS}"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Creat '.env'

```bash
sudo tee <<EOF >/dev/null $HOME/bundlr/validator-rust/.env
PORT=${BUNDLR_PORT}
VALIDATOR_KEY=./wallet.json
BUNDLER_URL=https://testnet1.bundlr.network
GW_WALLET=./wallet.json
GW_CONTRACT=RkinCLBlY4L5GZFv8gCFcrygTyd5Xm91CzKlR6qxhKA
GW_ARWEAVE=https://arweave.testnet1.bundlr.network
EOF
```

## Check

```bash
cat $HOME/bundlr/validator-rust/.env
```

## Run 'docker-compose'

```bash
cd $HOME/bundlr/validator-rust && docker-compose up -d
```

## Check logs

```bash
cd $HOME/bundlr/validator-rust && docker-compose logs -f --tail 10
```

## Build 'testnet-cli'

```bash
cd $HOME/bundlr/validator-rust && npm i -g @bundlr-network/testnet-cli
```

## Check balance wallet

```bash
cd $HOME/bundlr/validator-rust && testnet-cli balance ${BUNDLR_ADDRESS}
```

## Create a validator

```bash
cd $HOME/bundlr/validator-rust && \
testnet-cli join RkinCLBlY4L5GZFv8gCFcrygTyd5Xm91CzKlR6qxhKA \
-w ./wallet.json \
-u "http://$(wget -qO- eth0.me):${BUNDLR_PORT}" \
-s 25000000000000
```

## Service management

Restart docker
```bash
cd $HOME/bundlr/validator-rust && docker-compose restart
```

Stop docker

```bash
cd $HOME/bundlr/validator-rust && docker-compose stop
```

Run docker

```bash
cd $HOME/bundlr/validator-rust && docker-compose up -d
```

Delete docker

```bash
cd $HOME/bundlr/validator-rust && docker-compose down -v
```

Check logs

```bash
cd $HOME/bundlr/validator-rust && docker-compose logs -f --tail 10
```

View wallet address

```bash
cd $HOME/bundlr/validator-rust && \
cargo run --bin wallet-tool show-address \
--wallet wallet.json | jq ".address" | tr -d '"'
```

or

```bash
echo ${BUNDLR_ADDRESS}
```

View wallet balance

```bash
cd $HOME/bundlr/validator-rust && testnet-cli balance ${BUNDLR_ADDRESS}
```

## Update

```bash
cd $HOME/bundlr/validator-rust && \
git pull origin master && \
docker-compose up --build -d
```

## Delete Node

```bash
cd $HOME/bundlr/validator-rust && docker-compose down -v && \
cd $HOME
rm -Rvf $HOME/bundlr
```
