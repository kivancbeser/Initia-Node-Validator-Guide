# Initia-Node-Validator-Guide
Initia Node Validator Guide

### VALIDATOR TASK 
https://initia-xyz.notion.site/The-Initiation-Validator-Tasks-6d88ab0034644473907435662f9285b3

### OFFICIAL DOCUMENTATION
https://docs.initia.xyz/run-initia-node/running-initia-node/becoming-a-validator

### FAUCET 
https://faucet.testnet.initia.xyz/

### EXPLORER 
https://scan.testnet.initia.xyz/initiation-1

```
NOTE: ACTIVE SET IS JUST 30 VALIDATORS SO IF YOU CREATE YOU ARE JOINING THE INACTIVE LIST.
NOTE: DON'T FORGET TO BACKUP YOUR WALLET KEY AND priv_validator_key.json (in config)
```
### REQUIREMENTS 

| COMPONENTS | MINIMUM REQUIREMENTS | 
| ------------ | ------------ |
| CPU |	4|
| RAM	| 8+ GB |
| Storage	| 1000 GB SSD |
| System	| Ubuntu 22.04 |

### UPDATE UPGRADE MACHINE
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget build-essential jq make lz4 gcc unzip -y
```

### INSTALLATION GO
```
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

### MAKE INSTALL BINARY
```
git clone https://github.com/initia-labs/initia
cd initia
git checkout v0.2.15
make install
```

### DON'T FORGET TO WRITE YOUR MONIKER NAME 
```
initiad init "WRITE YOUR MONIKER NAME" --chain-id initiation-1
```

### GENESIS JSON
```
rm ~/.initia/config/genesis.json
wget https://initia.s3.ap-southeast-1.amazonaws.com/initiation-1/genesis.json
cp genesis.json ~/.initia/config/genesis.json
sed -i -e 's/external_address = \"\"/external_address = \"'$(curl httpbin.org/ip | jq -r .origin)':26656\"/g' ~/.initia/config/config.toml
```

### SET MINIMUM GAS PRICE
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.15uinit,0.01uusdc\"/" $HOME/.initia/config/app.toml
```

### PEERS SEEDS
```
PEERS="40d3f977d97d3c02bd5835070cc139f289e774da@168.119.10.134:26313,841c6a4b2a3d5d59bb116cc549565c8a16b7fae1@23.88.49.233:26656,e6a35b95ec73e511ef352085cb300e257536e075@37.252.186.213:26656,2a574706e4a1eba0e5e46733c232849778faf93b@84.247.137.184:53456,ff9dbc6bb53227ef94dc75ab1ddcaeb2404e1b0b@178.170.47.171:26656,edcc2c7098c42ee348e50ac2242ff897f51405e9@65.109.34.205:36656,07632ab562028c3394ee8e78823069bfc8de7b4c@37.27.52.25:19656,028999a1696b45863ff84df12ebf2aebc5d40c2d@37.27.48.77:26656,140c332230ac19f118e5882deaf00906a1dba467@185.219.142.119:53456,1f6633bc18eb06b6c0cab97d72c585a6d7a207bc@65.109.59.22:25756,065f64fab28cb0d06a7841887d5b469ec58a0116@84.247.137.200:53456,767fdcfdb0998209834b929c59a2b57d474cc496@207.148.114.112:26656,093e1b89a498b6a8760ad2188fbda30a05e4f300@35.240.207.217:26656,12526b1e95e7ef07a3eb874465662885a586e095@95.216.78.111:26656" && \
SEEDS="2eaa272622d1ba6796100ab39f58c75d458b9dbc@34.142.181.82:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@testnet-seeds.whispernode.com:25756" && \
sed -i \
    -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" \
    -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" \
    "$HOME/.initia/config/config.toml"
```

### PRUNING
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.initia/config/app.toml
```

### CREATE A SERVICE FILE
```
sudo tee /etc/systemd/system/initiad.service > /dev/null <<EOF
[Unit]
Description=initiad
After=network-online.target

[Service]
User=$USER
ExecStart=$(which initiad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### DAEMON RELOAD AND START SERVICE
```
sudo systemctl daemon-reload
sudo systemctl enable initiad  
sudo systemctl restart initiad  && sudo journalctl -fu initiad  -o cat
```

### CREATE NEW WALLET
DON'T FORGET TO SAVE YOUR KEYS
```
initiad keys add wallet
```

### GET FAUCET 
https://faucet.testnet.initia.xyz/

### RECOVERY WALLET
IF YOU REINSTALL NEW MACHINE WITH OLD WALLET USE THIS CODE FOR RECOVERY
```
initiad keys add wallet --recover
```

### WALLET LIST
```
initiad keys list
```

### CHECK SYNC
Before Create validator command, check your sync with this code. 
If shows false, run the create validator command.
```
initiad status 2>&1 | jq
```

### WALLET BALANCE CHECK
```
initiad q bank balances $(initiad keys show wallet -a)
```

### CREATE VALIDATOR (CHANGE MONIKERNAME)
```
initiad tx mstaking create-validator \
  --amount=9800000uinit \
  --pubkey=$(initiad tendermint show-validator) \
  --moniker="WRITE YOUR MONIKER NAME" \
  --chain-id=initiation-1 \
  --commission-rate=0.05 \
  --commission-max-rate=0.20 \
  --commission-max-change-rate=0.05 \
  --from=wallet \
  --identity="" \
  --website="" \
  --details="" \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 0.15uinit \
  -y
```
IF YOU ARE USING DIFFERNT PORT YOU NEED TO ADD THIS LINE CREATE VALIDATOR COMMAND. 
```
--node=http://localhost:PORTNUMBER \
```
FOR EXAMPLE:
--node=http://localhost:15657 \
!! PLEASE BACKUP YOUR PRIV VALID JSON

### EDIT VALIDATOR 
```
initiad tx mstaking edit-validator \
--moniker "NEW MONIKER NAME" \
--from cüzdan-adi-yaz \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.15uinit \
-y
```

### VALIDATOR INFO
```
initiad tx mstaking validator $(initiad keys show $WALLET --bech val -a)
```

### DELEGATION TO YOUR VALIDATOR
```
initiad tx mstaking delegate VALOPER_ADDRESS 1000000uinit --from wallet --gas=2000000 --fees=300000uinit --chain-id initiation-1 -y 
```

### UNJAIL
```
initiad tx slashing unjail --from wallet --fees=0.025uinit -y
```

### RESTART SERVICE
```
sudo systemctl restart initiad
```

### STOP SERVICE
```
sudo systemctl stop initiad
```

### CHECK LOG
```
sudo journalctl -u initiad -f -o cat
```

### PORT UPDATE
```
CUSTOM_PORT=110
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.initia/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.initia/config/app.toml
```


### REMOVING NODE
DON'T FORGET TO BACKUP YOUR PRIV VALIDATOR JSON KEY FROM CONFIG FOLDER.
```
cd $HOME
sudo systemctl stop initiad.service
sudo systemctl disable initiad.service
sudo rm /etc/systemd/system/initiad.service
sudo systemctl daemon-reload
rm -rf $HOME/.initia $HOME/initia
```

