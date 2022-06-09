#!/bin/bash
exists()
{
  command -v "$1" >/dev/null 2>&1
}
if exists curl; then
echo ''
else
  sudo apt update && sudo apt install curl -y < "/dev/null"
fi
bash_profile=$HOME/.bash_profile
if [ -f "$bash_profile" ]; then
    . $HOME/.bash_profile
fi
sleep 1 && curl -s https://databox.acadao.org/logo.sh | bash && sleep 1


if [ ! $NODENAME ]; then
read -p "Node isminiz: " NODENAME
echo 'export NODENAME='\"${NODENAME}\" >> $HOME/.bash_profile
fi
echo 'source $HOME/.bashrc' >> $HOME/.bash_profile
. $HOME/.bash_profile
sleep 1
cd $HOME
echo "export CHAIN_ID=sei-testnet-2" >> $HOME/.bash_profile
sudo apt update
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop -y < "/dev/null"

echo -e '\n\e[42mGo\e[0m\n' && sleep 1
cd $HOME
wget -O go1.18.1.linux-amd64.tar.gz https://golang.org/dl/go1.18.1.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && rm go1.18.1.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version

echo -e '\n\e[42mSEI\e[0m\n' && sleep 1
rm -rf $HOME/sei-chain
git clone --depth 1 --branch 1.0.2beta https://github.com/sei-protocol/sei-chain.git
cd sei-chain && make install
go build -o build/seid ./cmd/seid
chmod +x ./build/seid && sudo mv ./build/seid /usr/local/bin/seid

sleep 1

mv $HOME/go/bin/seid /usr/local/bin/
mv $HOME/sei-chain $HOME/sei
mv $HOME/.sei-chain $HOME/.sei

sleep 1


# config
seid config chain-id $CHAIN_ID
seid config keyring-backend file

# init
seid init $NODENAME --chain-id $CHAIN_ID

# download genesis and addrbook
wget -qO $HOME/.sei/config/genesis.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-testnet-2/genesis.json"
wget -qO $HOME/.sei/config/addrbook.json "https://raw.githubusercontent.com/sei-protocol/testnet/main/sei-testnet-2/addrbook.json"

# set minimum gas price
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usei\"/" $HOME/.sei/config/app.toml
# set peers and seeds
SEEDS=""
peers="257af61598dd3ce190bd7da84c6bcfeb5cbe9a99@rpc2.bonded.zone:21156"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sei/config/config.toml

# enable prometheus
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.sei/config/config.toml

# config pruning
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"

sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.sei/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.sei/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.sei/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.sei/config/app.toml

sleep 1

echo -e '\n\e[42mservis\e[0m\n' && sleep 1

seid unsafe-reset-all


# create service
tee $HOME/seid.service > /dev/null <<EOF
[Unit]
Description=seid
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which seid) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo mv $HOME/seid.service /etc/systemd/system/

# start service
sudo systemctl daemon-reload
sudo systemctl enable seid
sudo systemctl restart seid
echo -e "\e[1m\e[32m4 Kurulum tamamlandi... \e[0m" && sleep 1


echo -e '\n\e[42mNode Durumu\e[0m\n' && sleep 1
if [[ `service seid status | grep active` =~ "running" ]]; then
  echo -e "islem basarili \e[32mcalisiyor\e[39m!"
else
  echo -e "tekrar kur"
fi
