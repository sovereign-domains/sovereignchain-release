# Instructions for becoming a Sovereign validator.

**Minimum hardware spec:**  
2CPU, 4GB RAM, 50GB storage. N.B. As the network grows, all three requirements will increase.

**The following instructions apply for Ubuntu 22.04 LTS 64-bit**  
Throughout, replace `<moniker>` with the name of your node.

### 1. Update the local package list and install any available upgrades

```console
sudo apt-get update && sudo apt upgrade -y
```

### 2. Install toolchain and ensure accurate time synchronization

```console
sudo apt-get install make build-essential gcc git jq chrony -y
```

### 3. Add validator as system user

```console
sudo adduser <moniker>
```
```console
sudo usermod -aG sudo <moniker>
```

### 4. Log out and back in as the new user
```console
exit
```

### 5. Install Go

```console
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
```
```console
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz
```

### 6. Update profile
```console
pico ~/.profile
```


```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
export DAEMON_NAME=sovereignchaind
export DAEMON_HOME=$HOME/.sovereignchain
```

### 7. Apply the changes
```console
source ~/.profile
```

### 8. Check Go installation
```console
go version
````
```
go version go1.22.0 linux/amd64
```

### 9. Set up firewall
```console
sudo ufw default deny incoming &&
sudo ufw default allow outgoing &&
sudo ufw allow ssh &&
sudo ufw allow 26656/tcp &&
sudo ufw allow 26660/tcp
```

### 10. If the node would like to expose CometBFTs jsonRPC and Cosmos SDK GRPC and REST

```console
sudo ufw allow 26657/tcp &&
sudo ufw allow 9090/tcp &&
sudo ufw allow 1317/tcp
```

### 11. Enable firewall

```console
sudo ufw enable
```

### 12. Download and unzip Sovereign chain binary

```console
wget https://github.com/sovereign-domains/sovereignchain-release/raw/main/release/sovereignchain_linux_amd64.tar.gz
```
```console
tar xzf sovereignchain_linux_amd64.tar.gz
```

### 13. Initialize the application and create app key

```console
./sovereignchaind init <moniker> --chain-id sovereignchain
```
```console
./sovereignchaind keys add <moniker> --keyring-backend os
```
Store keyring passphrase and mnemonic somewhere safe.

### 14a. Copy config files and validate

```console
curl -LJ https://github.com/sovereign-domains/sovereignchain-release/raw/main/config/genesis.json -o ~/.sovereignchain/config/genesis.json
```
```console
chmod a-wx ~/.sovereignchain/config/genesis.json
```
```console
curl -LJ https://github.com/sovereign-domains/sovereignchain-release/raw/main/config/config.toml -o ~/.sovereignchain/config/config.toml
```
```console
curl -LJ https://github.com/sovereign-domains/sovereignchain-release/raw/main/config/app.toml -o ~/.sovereignchain/config/app.toml
```
```console
./sovereignchaind genesis validate-genesis
```

### 14b. Update config files manually

```console
ls ~/.sovereignchain/config/
```

```
app.toml  client.toml  config.toml  genesis.json  node_key.json  priv_validator_key.json
```

```console
pico ~/.sovereignchain/config/app.toml
```

```console
pico ~/.sovereignchain/config/config.toml
```

### 15. Back up keys

```console
~/.sovereignchain/config/node_key.json
~/.sovereignchain/config/priv_validator_key.json
```

### 16. Install Cosmovisor

```console
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```
```console
which cosmovisor
```

```
/home/<moniker>/go/bin/cosmovisor
```

### 17. Copy sovereignchain binary to Cosmovisor folder

```console
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin && mkdir -p $DAEMON_HOME/cosmovisor/upgrades &&
cp $HOME/sovereignchaind $DAEMON_HOME/cosmovisor/genesis/bin
```

### 18. Set up Cosmovisor service (remember to replace `<moniker>` throughout)

```console
sudo nano /etc/systemd/system/sovereignchaind.service
```

```
[Unit]
Description=Sovereign Daemon (cosmovisor)
After=network-online.target

[Service]
User=<moniker>
ExecStart=/home/<moniker>/go/bin/cosmovisor run start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=sovereignchaind"
Environment="DAEMON_HOME=/home/<moniker>/.sovereignchain"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"

[Install]
WantedBy=multi-user.target
```

```console
sudo -S systemctl daemon-reload &&
sudo -S systemctl enable sovereignchaind &&
sudo systemctl start sovereignchaind &&
sudo systemctl status sovereignchaind
```

```
     sovereignchaind.service - Sovereign Daemon (cosmovisor)
     Loaded: loaded (/etc/systemd/system/sovereignchaind.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-04-01 08:51:22 UTC; 23ms ago
   Main PID: 12667 (cosmovisor)
      Tasks: 6 (limit: 4642)
     Memory: 5.7M
        CPU: 19ms
     CGroup: /system.slice/sovereignchaind.service
             └─12667 /home/genesis/go/bin/cosmovisor run start

~$ systemd[1]: Started Sovereign Daemon (cosmovisor).
```

### 19. Check status

```console
./sovereignchaind status
```
```
{"node_info":{"protocol_version":{"p2p":"8","block":"11","app":"0"},"id":"f5ae61f6a8e0569c1d75beb7711efa89c7d32c66","listen_addr":"tcp://0.0.0.0:26656","network":"sovereignchain","version":"0.38.5","channels":"40202122233038606100","moniker":"genesis","other":{"tx_index":"on","rpc_address":"tcp://0.0.0.0:26657"}},"sync_info":{"latest_block_hash":"35F36E10CCA14A6E785F1693C25B85265F5A5461D17C52EA5787D3314514C042","latest_app_hash":"539662503C543FB02DD25EC53A00309BC71475C8ACD0B28D9E96D0385019DEC8","latest_block_height":"10981","latest_block_time":"2024-04-02T16:52:04.157878138Z","earliest_block_hash":"44EB8259843BDA9FD40A7F6783C4DCFD703C1A065397D2E718420778E1A87543","earliest_app_hash":"E3B0C44298FC1C149AFBF4C8996FB92427AE41E4649B934CA495991B7852B855","earliest_block_height":"1","earliest_block_time":"2024-04-02T13:40:30.78060932Z","catching_up":false},"validator_info":{"address":"C1E3A5A1269EA4FAE654D6F624A51290C35E5890","pub_key":{"type":"tendermint/PubKeyEd25519","value":"MMjU0LekWnovFrdGJly7JcW1/xhSiwGIaqXWQESiGyE="},"voting_power":"0"}}
```

### 20. Monitor logs (CTRL+C to exit)

```console
journalctl -fu sovereignchaind
```
