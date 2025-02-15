<div>
<h1 align="left" style="display: flex;"> NYM Mixnode Setup for Mainnet</h1>
<img src="https://avatars.githubusercontent.com/u/51752891?s=200&v=4"  style="float: right;" width="100" height="100"></img>
</div>

Official documentation:
>- [Validator setup instructions](https://nymtech.net/docs/stable/run-nym-nodes/nodes/mixnodes)

Explorer:
>-  https://explorer.nymtech.net/network-components/mixnodes


## Hardware Requirements
### Recommended Hardware Requirements 
 - 2 vCPU
 - 1GB RAM
 - Support of IPv4 and IPv6

## Set up your Nym Mix node
>To start with Nym mixnode installation please first install and create your [Nym wallet](https://nymtech.net/download/)

Update packages and Install dependencies

~~~bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux pkg-config libssl-dev build-essential jq make lz4 gcc -y
~~~

Replace your wallet and moniker `<YOUR_WALLET_ADDRESS>` `<NYM_MONIKER>` without `<>`, save and import variables into system

~~~bash
echo "export NYM_WALLET_ADDRESS="<YOUR_WALLET_ADDRESS>"" >> $HOME/.bash_profile
echo "export NYM_MONIKER="<YOUR_MONIKER>"" >> $HOME/.bash_profile
source $HOME/.bash_profile
~~~

install go

~~~bash
cd $HOME
VER="1.18.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm -rf  "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
~~~

Install Rust

~~~bash
apt install curl -y
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
~~~

`Press 1`
Install Cargo 

~~~bash
sudo apt install cargo -y
source ~/.cargo/env 
rustup update stable
~~~

Download and build binaries

~~~bash
cd $HOME
rm -rf nym
git clone https://github.com/nymtech/nym.git
cd nym
git checkout $(curl -sSL 'https://api.github.com/repos/nymtech/nym/releases' | grep nym-mixnodes | grep -E -o "nym-binaries-v[0-9]\.]?[0-9]\.]?[0-9]\]?" | sort | tail -n 1)
cargo build --bin nym-client --release
~~~

Move binaries, add permissions 
~~~bash
mv $HOME/nym/target/debug/nym-mixnode $HOME/go/bin/nym-mixnode
chmod u+x $HOME/go/bin/nym-mixnode
~~~

>You can check that your binaries are properly compiled
~~~bash
nym-mixnode --help
~~~

Config and init app

~~~bash
nym-mixnode init --id $NYM_MONIKER --host $(curl ifconfig.me) --wallet-address $NYM_WALLET_ADDRESS
~~~

## Bonding your mix node
### You can bond your mix node via the Desktop wallet.

>Open your wallet, and head to the Bond page, then select the node type and input your node details: 
`Identity Key` `Sphinx Key` `Owner Signature` `Host` `Version` and bond your mixnode

~~~bash
nym-mixnode node-details --id $NYM_MONIKER
~~~

Create Service file

~~~bash
sudo tee /etc/systemd/system/nym-mixnode.service > /dev/null <<EOF
[Unit]
Description=Nym Mixnode
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=$USER
LimitNOFILE=65536
ExecStart=$(which nym-mixnode) run --id $NYM_MONIKER
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
EOF
~~~

Enable and start service

~~~bash
sudo systemctl daemon-reload
sudo systemctl enable nym-mixnode
sudo systemctl start nym-mixnode && sudo journalctl -u nym-mixnode -f
~~~

(Optional) describe your mix node

~~~bash
nym-mixnode describe --id $NYM_MONIKER
~~~

## To check mixed packets

~~~bash
sudo journalctl -u nym-mixnode -f | grep "Since startup mixed"
~~~
>If you don't see any mixed packages, just repeat it after a while again

### Security
Please don`t share your privkey, mnemonic and follow a basic security rules

### Set up ssh keys for authentication
You can use this [guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04) to configure ssh authentication and disable password authentication on your server

### Firewall security
Set the default to allow outgoing connections, deny all incoming, allow ssh and node p2p port

~~~bash
sudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow 1789,1790,8000/tcp
sudo ufw enable
~~~

## Usefull commands
### Service commands
check logs

~~~bash
sudo journalctl -u nym-mixnode -f
~~~

stop service

~~~bash
sudo systemctl stop nym-mixnode
~~~

start service

~~~bash
sudo systemctl start nym-mixnode
~~~

restart service

~~~bash
sudo systemctl restart nym-mixnode
~~~

To check mixed packets

~~~bash
sudo journalctl -u nym-mixnode -f | grep "Since startup mixed"
~~~
>If you don't see any mixed packages, just repeat it after a while again

### Delete node

~~~bash
sudo systemctl stop nym-mixnode
sudo systemctl disable nym-mixnode
sudo rm -rf /etc/systemd/system/nym-mixnode*
sudo rm $(which nym-mixnode)
sudo rm -rf $HOME/.nym
sudo rm -fr $HOME/nym
sed -i "/NYM_/d" $HOME/.bash_profile
~~~

