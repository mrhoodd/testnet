# Install Guide Testnet v2

## [Website](https://elixir.finance/) | [Twitter](https://twitter.com/ElixirProtocol) | [Discord](https://discord.gg/GeKAZFjYT3) | :satellite:[Explorer](https://dashboard.elixir.finance/)

## Install docker

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

## Create a directory

```bash
mkdir elixir
```

```bash
cd elixir
```

## Download the Dockerfile

```bash
curl -O https://files.elixir.finance/Dockerfile
```

## Edit Dockerfile

:red_circle:Enter the metamask wallet address in ENV ADDRESS and the private key in ENV PRIVATE_KEY

```bash
nano Dockerfile
```

## Build the Docker image

```bash
docker build . -f Dockerfile -t elixir-validator
```

## Start your validator

```bash
docker run -d --restart unless-stopped --name ev elixir-validator
```

## Getting the latest version

```bash
cd elixir
docker kill ev
docker rm ev
docker pull elixirprotocol/validator:testnet-2
docker build . -f Dockerfile -t elixir-validator
```

## And rerun the docker command to start the validator:

```bash
docker run -d --restart unless-stopped --name ev elixir-validator
```
