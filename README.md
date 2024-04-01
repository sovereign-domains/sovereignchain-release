# Instructions for becoming a Sovereign validator.

**Minimum hardware spec:**  
2CPU, 4GB RAM, 50GB storage. N.B. As the network grows, all three requirements will increase.

**The following instructions apply for Ubuntu 22.04 LTS 64-bit**  
Information inside <> should be replaced with your data. For example, if you are naming your node 'mynode', <moniker> should be replaced with 'mynode'.  

**Update the local package list and install any available upgrades**  
```console
~$ sudo apt-get update && sudo apt upgrade -y
```

**Install toolchain and ensure accurate time synchronization**  
$ sudo apt-get install make build-essential gcc git jq chrony -y  

**Add validator as system user**  
$ sudo adduser <moniker>  
$ sudo usermod -aG sudo <moniker>  

**Log out and back in as the new user**

**Install Go**
$ wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
$ sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

**Update profile**
$ export GOROOT=/usr/local/go
$ export GOPATH=$HOME/go
$ export GO111MODULE=on
$ export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
$ export DAEMON_NAME=sovereignchaind
$ export DAEMON_HOME=$HOME/.sovereignchain

$ source ~/.profile 

**Log out and back in**
**Check Go installation**
$ go version
should return something like this: go version go1.22.0 linux/amd64

**Set up firewall**
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow ssh
$ sudo ufw allow 26656/tcp
$ sudo ufw allow 26660/tcp

**If the node would like to expose CometBFTs jsonRPC and Cosmos SDK GRPC and REST**
$ sudo ufw allow 26657/tcp
$ sudo ufw allow 9090/tcp
$ sudo ufw allow 1317/tcp

$ sudo ufw enable

**Download binary**
$ curl -O https://github.com/sovereign-domains/sovereignchain-release/raw/main/release/sovereignchain_linux_amd64.tar.gz

