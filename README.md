
![1500x500](https://github.com/user-attachments/assets/bd8092af-aefe-4374-a52a-e3c9e219df11)

<h1 align="center"> AtomOne


</h1>



## 💻 Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	4|
| RAM	| 8+ GB |
| Storage	| 512 GB SSD |



### 💢Required Installations
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip gcc clang cmake build-essential -y
```

### 💢 Go Install
```
cd $HOME
VER="1.23.0"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### 💢Set Vars
```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export ATOMONE_CHAIN_ID="atomone-1"" >> $HOME/.bash_profile
echo "export ATOMONE_PORT="17"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### 💢Download Binary
```
cd $HOME
rm -rf atomone
git clone https://github.com/atomone-hub/atomone
cd atomone
git checkout v1.0.0
make install
```

### 💢Configuration settings
```
atomoned init $MONIKER --chain-id $ATOMONE_CHAIN_ID
sed -i \
-e "s/chain-id = .*/chain-id = \"atomone-1\"/" \
-e "s/keyring-backend = .*/keyring-backend = \"os\"/" \
-e "s/node = .*/node = \"tcp:\/\/localhost:${ATOMONE_PORT}657\"/" $HOME/.atomone/config/client.toml
```

### 💢Genesis and addrbook
```
wget -O $HOME/.atomone/config/genesis.json https://server-7.itrocket.net/mainnet/atomone/genesis.json
wget -O $HOME/.atomone/config/addrbook.json  https://server-7.itrocket.net/mainnet/atomone/addrbook.json
```
### 💢Seed and Peers
```
SEEDS="f19d9e0f8d48119aa4cafde65de923ae2c29181a@atomone-mainnet-seed.itrocket.net:61656"
PEERS="ed0e36c57122184ab05b6c635b2f2adf592bfa0c@atomone-mainnet-peer.itrocket.net:61657,e6ccf37182345ad048cf60335ab3dc1a5512baff@13.59.146.253:26656,d3adcf9eee8665ee2d3108f721b3613cdd18c3a3@23.227.223.49:26656,2a3530e9778122cf9301fb034f6c92d9842049d3@46.166.143.72:26656,5ad3d484730844e66f15926c4fcc006c77b53ddd@88.99.137.151:26656,5d913650738a081aa02631a7f108dc7812330f0b@37.27.129.24:13656,42c384bdf78ea2a2e7fc0c4e1716ef94951fca16@95.214.52.233:36656,4ef48d2cc03b332f9a711fc65dc0453839f9040d@8.52.153.92:61656,6c4b686add2ae26aad617a15e4db012e7496eee1@154.91.1.108:26656,6219193bd3b127e980686b745b97bdcd2ff429ef@95.217.107.137:11656,9bb19453ef776cfb09085f4d9465d5073f2484f4@142.132.140.178:29956"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.atomone/config/config.toml
```
### 💢Custom Ports app.toml
```
sed -i.bak -e "s%:1317%:${ATOMONE_PORT}317%g;
s%:8080%:${ATOMONE_PORT}080%g;
s%:9090%:${ATOMONE_PORT}090%g;
s%:9091%:${ATOMONE_PORT}091%g;
s%:8545%:${ATOMONE_PORT}545%g;
s%:8546%:${ATOMONE_PORT}546%g;
s%:6065%:${ATOMONE_PORT}065%g" $HOME/.atomone/config/app.toml
```
### 💢Custom Ports config.toml
```
sed -i.bak -e "s%:26658%:${ATOMONE_PORT}658%g;
s%:26657%:${ATOMONE_PORT}657%g;
s%:6060%:${ATOMONE_PORT}060%g;
s%:26656%:${ATOMONE_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ATOMONE_PORT}656\"%;
s%:26660%:${ATOMONE_PORT}660%g" $HOME/.atomone/config/config.toml
```
### 💢Config Pruning
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.atomone/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.atomone/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.atomone/config/app.toml
```
### 💢Set minimum gas price
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.001uatone"|g' $HOME/.atomone/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.atomone/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.atomone/config/config.toml
```

### 💢Servis Setup
```
sudo tee /etc/systemd/system/atomoned.service > /dev/null <<EOF
[Unit]
Description=Atomone node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.atomone
ExecStart=$(which atomoned) start --home $HOME/.atomone
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable zenrockd.service
```
### 💢Download Snapshot
```
atomoned tendermint unsafe-reset-all --home $HOME/.atomone
if curl -s --head curl https://server-7.itrocket.net/mainnet/atomone/atomone_2025-02-05_1665623_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://server-7.itrocket.net/mainnet/atomone/atomone_2025-02-05_1665623_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.atomone
    else
  echo "no snapshot found"
fi
```
### 💢Enable and Start Node 
```
sudo systemctl daemon-reload
sudo systemctl enable atomoned
sudo systemctl restart atomoned && sudo journalctl -u atomoned -fo cat
```

### 💢Create wallet or Import wallet
Create wallet
```
atomoned keys add $WALLET
```
Import wallet
```
atomoned keys add $WALLET --recover
```

### 💢Create Validator

```
atomoned tx staking create-validator \
  --amount 1000000uatone \
  --from $WALLET \
  --commission-rate 0.1 \
  --commission-max-rate 0.2 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --pubkey $(atomoned tendermint show-validator) \
  --moniker "moniker-name" \
  --identity "" \
  --website "" \
  --details "" \
  --chain-id atomone-1 \
  --gas 500000 \
  --fees 500uatone \
  -y

```
### 💢Delegate Validator

```
atomoned tx staking delegate $(atomoned keys show $WALLET --bech val -a) 1000000uatone --from $WALLET --chain-id atomone-1 --gas 500000 --fees 500uatone -y
```
### 💢Sync Status
```
atomoned status 2>&1 | jq 
```
### 💢Delete Node
```
sudo systemctl stop atomoned
sudo systemctl disable atomoned
sudo rm -rf /etc/systemd/system/atomoned.service
sudo rm $(which atomoned)
sudo rm -rf $HOME/.atomone
sed -i "/ATOMONE_/d" $HOME/.bash_profile
```

```
Thanks Itrocket...
```


