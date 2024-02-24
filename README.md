# Babylon-3-testnet (Обратите внимание - изменились требования к железу)



#### Discord https://discord.gg/babylonglobal
### Babylon - это новый проект Cosmos, целью которого является использование безопасности Bitcoin для повышения безопасности зон Cosmos и других цепочек PoS. 
#### Ссылка на команду https://babylonchain.io/about
#### Сайт: https://babylonchain.io/foundation
#### Твиттер: https://www.twitter.com/babylon_chain
#### GitHub https://github.com/babylonchain/babylonchain.github.io
#### Zealy https://zealy.io/c/babylonchain/invite/1zn87lyrLTOaCWHZgpHR8

### Требования к серверу 

#### Recommended 4CPU 32RAM 1000GB

## как вариант заказать здесь https://powervps.net/ru/?from=91820


### удалить старую версию

```
sudo systemctl stop babylond && sudo systemctl disable babylond && sudo rm /etc/systemd/system/babylond.service && sudo systemctl daemon-reload && rm -rf $HOME/.babylond && rm -rf babylon && sudo rm -rf $(which babylond) 
```


## Обновление пакетов сервера и подготовка к развертыванию ноды
```
sudo apt update && sudo apt upgrade -y
```

Setup Validator Name
First change “Имя вашего валидатора” to your chosen validator moniker and enter this command:
```
MONIKER="Имя_вашего_валидатора"
```
Install Dependencies & Install GO
Install Build Tools
```
sudo apt -qy install curl git jq lz4 build-essential
```
Install GO
```
ver="1.22.0"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

Download And Build Binaries
```
# Clone project repository
cd $HOME
rm -rf babylon
git clone https://github.com/babylonchain/babylon.git
cd babylon
git checkout v0.8.3
```
```
# Build binaries
make build
```
```
# Prepare binaries for Cosmovisor
mkdir -p ~/.babylond
mkdir -p ~/.babylond/cosmovisor
mkdir -p ~/.babylond/cosmovisor/genesis
mkdir -p ~/.babylond/cosmovisor/genesis/bin
mkdir -p ~/.babylond/cosmovisor/upgrades
```
```
mv build/babylond $HOME/.babylond/cosmovisor/genesis/bin/
rm -rf build
```
```
# Create application symlinks
sudo ln -s $HOME/.babylond/cosmovisor/genesis $HOME/.babylond/cosmovisor/current -f
sudo ln -s $HOME/.babylond/cosmovisor/current/bin/babylond /usr/local/bin/babylond -f
```

Set Up Cosmovisor And Create The Corresponding Service
```
# Download and install Cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```
```
# Create and start service
sudo tee /etc/systemd/system/babylond.service > /dev/null <<EOF
[Unit]
Description=Babylon daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --x-crisis-skip-assert-invariants
Restart=always
RestartSec=3
LimitNOFILE=infinity

Environment="DAEMON_NAME=babylond"
Environment="DAEMON_HOME=${HOME}/.babylond"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"

[Install]
WantedBy=multi-user.target
EOF
```

## Меняем порты на нестандартные
```
sed -i.bak -e "s%:26658%:35658%; s%:26657%:35657%; s%:6060%:6960%; s%:26656%:35656%; s%:26660%:35660%" $HOME/.babylond/config/config.toml && sed -i.bak -e "s%:9090%:9990%; s%:9091%:9991%; s%:1317%:2217%; s%:8545%:9445%; s%:8546%:9446%; s%:6065%:6965%" $HOME/.babylond/config/app.toml && sed -i.bak -e "s%:26657%:35657%" $HOME/.babylond/config/client.toml 
```

```
sudo systemctl daemon-reload
sudo systemctl enable babylond.service
```


Initialize The Node
```
# Initialize the node
babylond init $MONIKER --chain-id bbn-test-3
```
```
# Add seeds
sed -i -e "s|^seeds *=.*|seeds = \"49b4685f16670e784a0fe78f37cd37d56c7aff0e@3.14.89.82:26656,9cb1974618ddd541c9a4f4562b842b96ffaf1446@3.16.63.237:26656\"|" $HOME/.babylond/config/config.toml

# Set minimum gas price
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.00001ubbn\"|" $HOME/.babylond/config/app.toml

# Switch to signet
sed -i -e "s|^network *=.*|network = \"signet\"|" $HOME/.babylond/config/app.toml

# Set pruning
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.babylond/config/app.toml
```

Retrieve The Genesis File:
```
wget https://github.com/babylonchain/networks/raw/main/bbn-test-3/genesis.tar.bz2
tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
mv genesis.json ~/.babylond/config/genesis.json
```
Start Service And Check The Logs
```
sudo systemctl start babylond.service && sudo journalctl -u babylond.service -f --no-hostname -o cat
```

### если не коннектится можно взять пиры сдесь

```
https://www.polkachu.com/testnets/babylon/peers
```
```
https://nodestake.org/babylon
```
### скачать снап

```
https://nodestake.org/babylon
```


Becoming a Validator

```
# Create a New Key
babylond keys add wallet
```
или восстанавливаем прошлый
```
babylond keys add wallet --recover
```
вводим сид фразу от кошелька

Obtain Funds from the Babylon Testnet Faucet

### идем в дискорд и запрашиываем токены

!faucet адрес кошелька

You can check your wallet balance using this command:

### проверяем их на кошельке
```
babylond q bank balances $(babylond keys show wallet -a)
```
Ensure that you have successfully received 1,100,000 ubbn. Please be aware that you can only claim 1 BBN from the faucet every 24 hours.


Generate a BLS Key Pair
```
babylond create-bls-key $(babylond keys show wallet -a)
```



```
sed -i -e "s|^key-name *=.*|key-name = \"wallet\"|" $HOME/.babylond/config/app.toml
```

```
sed -i -e "s|^timeout_commit *=.*|timeout_commit = \"10s\"|" $HOME/.babylond/config/config.toml
```

### рестартим и проверяем логи

```
sudo systemctl daemon-reload
```
```
sudo systemctl start babylond.service && sudo journalctl -u babylond.service -f --no-hostname -o cat
```


Create the Validator

```
babylond status | jq .SyncInfo
```

### После синхронизации создаем файл validator.json
```
{
        "pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"SBLBy6fLwg4TZFqTdT3muxffkTj9YlsIyPB3L8oYg="},
        "amount": "1000000ubbn",
        "chain-id": "bbn-test-3",
        "moniker": "Ваш_моникер",
        "website": "@WingsNodeTeam",
        "security": "telegramm @WingsNodeTeam",
        "details": "-_-",
        "commission-rate": "0.1",
        "commission-max-rate": "0.2",
        "commission-max-change-rate": "0.01",
        "min-self-delegation": "1"
}
```
где, pubkey можно узнать командой 
```
babylond tendermint show-validator
```
### сохраняем и записываем путь к этому файлу, пример 
/root/.babylond/config/validator.json

### создаем валидатора

```
babylond tx checkpointing create-validator /root/.babylond/config/validator.json --from=адрес_кошелька --chain-id bbn-test-3 --fees=12000ubbn -y
```
### ждем 1 час и проверяем валидатора в браузере
```
https://testnet.babylon.explorers.guru/validators
```

### делегировать себе
babylond tx epoching delegate адрес_валидатора 1000000ubbn --from=ваш_кошелек --chain-id bbn-test-3 --fees=12000ubbn -y
```


