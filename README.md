**Dependencies Installation**

**Install dependencies for building from source**

```sudo apt update
sudo apt install -y curl git jq lz4 build-essential
```

**Install Go**
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
Node Installation
```

**Clone project repository**
```
cd && rm -rf canine-chain
git clone https://github.com/JackalLabs/canine-chain
cd canine-chain
git checkout v4.0.3
```

**Build binary**
```
make install
```

**Set node CLI configuration**
```
canined config chain-id jackal-1
canined config keyring-backend file
canined config node tcp://localhost:17557
```

**Initialize the node**
```
canined init "Your Node Name" --chain-id jackal-1
```

**Download genesis and addrbook files**
```
curl -L https://snapshots.nodejumper.io/jackal/genesis.json > $HOME/.canine/config/genesis.json
curl -L https://snapshots.nodejumper.io/jackal/addrbook.json > $HOME/.canine/config/addrbook.json
```

**Set seeds**
```

sed -i -e 's|^seeds *=.*|seeds = "20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:17556,ebc272824924ea1a27ea3183dd0b9ba713494f83@jackal-mainnet-seed.autostake.com:26906,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:17556,400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@jackal.rpc.kjnodes.com:13759,c28827cb96c14c905b127b92065a3fb4cd77d7f6@seeds.whispernode.com:17556,8542cd7e6bf9d260fef543bc49e59be5a3fa9074@seed.publicnode.com:26656,0ab9ec918cd36a28be1fcf538f7f76ede2b81659@89.58.38.59:26656,ebc272824924ea1a27ea3183dd0b9ba713494f83@jackal-mainnet-peer.autostake.com:26906,26b6255375a592c3b0664bd474a6975f468c3785@jkl.peer.stavr.tech:11126,713d202326eedaed41d467b26051aba62727febd@5.9.69.241:26656"|' $HOME/.canine/config/config.toml
```

**Set minimum gas price**
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.002ujkl"|' $HOME/.canine/config/app.toml
```

**Set pruning**
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.canine/config/app.toml
```

**Change ports**
```
sed -i -e "s%:1317%:17517%; s%:8080%:17580%; s%:9090%:17590%; s%:9091%:17591%; s%:8545%:17545%; s%:8546%:17546%; s%:6065%:17565%" $HOME/.canine/config/app.toml
sed -i -e "s%:26658%:17558%; s%:26657%:17557%; s%:6060%:17560%; s%:26656%:17556%; s%:26660%:17561%" $HOME/.canine/config/config.toml
```

**Download latest chain data snapshot**
```
curl "https://snapshots.nodejumper.io/jackal/jackal_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.canine"
```

**Create a service**
```
sudo tee /etc/systemd/system/canined.service > /dev/null << EOF
[Unit]
Description=Jackal node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which canined) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable canined.service
```

**Start the service and check the logs**
```
sudo systemctl start canined.service
sudo journalctl -u canined.service -f --no-hostname -o cat
Secure Server Setup (Optional)
```

**generate ssh keys, if you don't have them already, DO IT ON YOUR LOCAL MACHINE**
```
ssh-keygen -t rsa
```

**save the output, we'll use it later on instead of YOUR_PUBLIC_SSH_KEY**
```
cat ~/.ssh/id_rsa.pub
```

**upgrade system packages**
```
sudo apt update
sudo apt upgrade -y
```

**add new admin user**
```
sudo adduser admin --disabled-password -q
```

**upload public ssh key, replace YOUR_PUBLIC_SSH_KEY with the key above**
```
mkdir /home/admin/.ssh
echo "YOUR_PUBLIC_SSH_KEY" >> /home/admin/.ssh/authorized_keys
sudo chown admin: /home/admin/.ssh
sudo chown admin: /home/admin/.ssh/authorized_keys
```

echo "admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

**disable root login, disable password authentication, use ssh keys only**
```
sudo sed -i 's|^PermitRootLogin .*|PermitRootLogin no|' /etc/ssh/sshd_config
sudo sed -i 's|^ChallengeResponseAuthentication .*|ChallengeResponseAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PasswordAuthentication .*|PasswordAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PermitEmptyPasswords .*|PermitEmptyPasswords no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PubkeyAuthentication .*|PubkeyAuthentication yes|' /etc/ssh/sshd_config

sudo systemctl restart sshd
```

**install fail2ban**
```
sudo apt install -y fail2ban
```

**install and configure firewall**
```
sudo apt install -y ufw
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 9100
sudo ufw allow 26656
```

# make sure you expose ALL necessary ports, only after that enable firewall
sudo ufw enable

# make terminal colorful
sudo su - admin
source <(curl -s https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/enable_colorful_bash.sh)

# update servername, if needed, replace YOUR_SERVERNAME with wanted server name
sudo hostnamectl set-hostname YOUR_SERVERNAME

# now you can logout (exit) and login again using ssh admin@YOUR_SERVER_IP
