# Soarchain

### Soarchain node Installation Instructions.

[Official documentation](https://docs.soarchain.com)

System requirements:</br>
CPU: Quad Core or larger AMD or Intel (amd64) CPU
RAM:32GB RAM
SSD:1TB NVMe Storage
100MBps bidirectional internet connection
OS: Ubuntu 22.04</br>

You can take a weaker server

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>

### Installing the Babylon Node

1. Preparing the server/Required packages installation</br>
```
sudo apt update
sudo apt upgrade -y
sudo apt-get install libclang-dev
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```
### Go installation.
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd $HOME
curl -s https://raw.githubusercontent.com/soar-robotics/testnet-binaries/main/v0.2.10/ubuntu22.04/soarchaind > soarchaind
chmod +x soarchaind
mkdir -p $HOME/go/bin/
mv soarchaind $HOME/go/bin/
sudo wget -P /usr/lib https://github.com/CosmWasm/wasmvm/releases/download/v1.3.0/libwasmvm.x86_64.so
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

# Config and init app
```
soarchaind config chain-id soarchaintestnet
soarchaind config keyring-backend test
soarchaind config node tcp://localhost:25257
soarchaind init "your moniker" --chain-id soarchaintestnet
```

# Download genesis and addrbook
```
curl -L https://snapshots-testnet.nodejumper.io/soarchain-testnet/genesis.json > $HOME/.soarchain/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/soarchain-testnet/addrbook.json > $HOME/.soarchain/config/addrbook.json
```

# Set seeds and peers
```
sed -i -e 's|^seeds *=.*|seeds = "3f472746f46493309650e5a033076689996c8881@soarchain-testnet.rpc.kjnodes.com:17259"|' $HOME/.soarchain/config/config.toml

sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001utmotus"|' $HOME/.soarchain/config/app.toml

sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.soarchain/config/app.toml
```

# Create service file
```
sudo tee /etc/systemd/system/soarchaind.service > /dev/null << EOF
[Unit]
Description=Soarchain node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which soarchaind) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable soarchaind.service
```

# Reset and download snapshot
```
curl "https://snapshots-testnet.nodejumper.io/soarchain-testnet/soarchain-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.soarchain"
```

# enable and start service
```
sudo systemctl start soarchaind.service
sudo journalctl -u soarchaind.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
soarchaind keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
soarchaind keys add wallet --recover
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
crossfid status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'
```

### We receive tokens from the tap in the [discord](https://discord.gg/g6CvcgTt8b)
```
faucet not work
```

# before creating a validator, you need to fund your wallet and check balance
```
soarchaind q bank balances $(soarchaind keys show wallet -a) 
```
# Create validator
```
soarchaind tx staking create-validator \
--amount=1000000utmotus \
--pubkey=$(soarchaind tendermint show-validator) \
--moniker="your moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO❤️" \
--chain-id=soarchaintestnet \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.0001utmotus \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
sudo systemctl stop soarchaind

cd $HOME
curl -s https://raw.githubusercontent.com/soar-robotics/testnet-binaries/main/v0.2.10/ubuntu22.04/soarchaind > soarchaind
chmod +x soarchaind
mkdir -p $HOME/go/bin/
mv soarchaind $HOME/go/bin/

curl "https://snapshots-testnet.nodejumper.io/soarchain-testnet/soarchain-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.soarchaind"

sudo systemctl start soarchaind
sudo journalctl -u soarchaind -f --no-hostname -o cat

Current network:soarchaintestnet
Current version:v0.2.10
```

### Useful commands

Check balance
```
soarchaind q bank balances $(soarchaind keys show wallet -a) 
```

CHECK SERVICE LOGS
```
sudo journalctl -u soarchaind -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart soarchaind
```

GET VALIDATOR INFO
```
soarchaind status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
soarchaind tx staking delegate $(soarchaind keys show wallet --bech val -a) 1000000utmotus --from wallet --chain-id soarchaintestnet --gas-prices 0.0001utmotus --gas-adjustment 1.5 --gas auto -y 
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop soarchaind && sudo systemctl disable soarchaind && sudo rm /etc/systemd/system/soarchaind.service && sudo systemctl daemon-reload && rm -rf $HOME/.soarchain && rm -rf testnet-binaries && sudo rm -rf $(which soarchaind) 
```
