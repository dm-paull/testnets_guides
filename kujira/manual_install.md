# Manual node  setup
If you want to setup fullnode manually follow the steps below

## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<MY_MONIKER_NAME_GOES_HERE>
```

Save and import variables into system
```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export WALLET=wallet" >> $HOME/.bash_profile
echo "export CHAIN_ID=harpoon-4" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux -y
```

## Install go
```
ver="1.18.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

## Download and build binaries
```
cd $HOME
git clone https://github.com/Team-Kujira/core kujira-core && cd kujira-core
git checkout v0.4.0
make install
```

## Config app
```
kujirad config chain-id $CHAIN_ID
kujirad config keyring-backend file
```

## Init app
```
kujirad init $NODENAME --chain-id $CHAIN_ID
```

## Download genesis and addrbook
```
wget -qO $HOME/.kujira/config/genesis.json "https://raw.githubusercontent.com/Team-Kujira/networks/master/testnet/harpoon-4.json"
```

## Set minimum gas price and timeout commit
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00125ukuji\"/" $HOME/.kujira/config/app.toml
sed -i -e "s/^timeout_commit *=.*/timeout_commit = \"1500ms\"/" $HOME/.kujira/config/config.toml
```

## Set seeds and peers
```
SEEDS=""
PEERS="87ea1a43e7eecdd54399551b767599921e170399@52.215.221.93:26656,021b782ba721e799cd3d5a940fc4bdad4264b148@65.108.103.236:16656,1d6f841271a1a3f78c6772b480523f3bb09b0b0b@15.235.47.99:26656,ccd2861990a98dc6b3787451485b2213dd3805fa@185.144.99.234:26656,909b8da1ea042a75e0e5c10dc55f37711d640388@95.216.208.150:53756,235d6ac8aebf5b6d1e6d46747958c6c6ff394e49@95.111.245.104:26656,b525548dd8bb95d93903b3635f5d119523b3045a@194.163.142.29:26656,26876aff0abd62e0ab14724b3984af6661a78293@139.59.38.171:36347,21fb5e54874ea84a9769ac61d29c4ff1d380f8ec@188.132.128.149:25656,06ebd0b308950d5b5a0e0d81096befe5ba07e0b3@193.31.118.143:25656,f9ee35cf9aec3010f26b02e5b3354efaf1c02d53@116.203.135.192:26656,c014d76c1a0d1e0d60c7a701a7eff5d639c6237c@157.90.179.182:29656,0ae4b755e3da85c7e3d35ce31c9338cb648bba61@164.92.187.133:26656,202a3d8bd5a0e151ced025fc9cbff606845c6435@49.12.222.155:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.kujira/config/config.toml
```

## Disable indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.kujira/config/config.toml
```

## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.kujira/config/config.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.kujira/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.kujira/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.kujira/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.kujira/config/app.toml
```

## Reset chain data
```
kujirad tendermint unsafe-reset-all
```

## Create service
```
sudo tee /etc/systemd/system/kujirad.service > /dev/null <<EOF
[Unit]
Description=kujira
After=network-online.target

[Service]
User=$USER
ExecStart=$(which kujirad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable kujirad
sudo systemctl restart kujirad && sudo journalctl -u kujirad -f -o cat
```
