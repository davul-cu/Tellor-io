# Tellor-io
1. Install Necessary Dependencies
```
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential git curl jq lz4 net-tools -y
```
2. Install GO
```
wget https://golang.org/dl/go1.21.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.3.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```
3. Install Node
```
git clone --branch v3.0.3 https://github.com/tellor-io/layer $HOME/layer
cd $HOME/layer
make install
```
4.Install Cosmovisor(for auto-upgrade)
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
export PATH=$PATH:$HOME/go/bin
```
5.Configuration
```
layerd init "your-monıker-name" --chain-id layertest-3
mkdir -p $HOME/.layer/cosmovisor/data
```
6.Create Service File 
```
sudo tee /etc/systemd/system/layerd.service <<EOF
[Unit]
Description=Tellor Node
After=network-online.target
[Service]
User=$USER
Environment="DAEMON_NAME=layerd"
Environment="DAEMON_HOME=$HOME/.layer"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/root/.layer/cosmovisor/current/bin"
Environment="ETH_RPC_URL=wss://a.good.sepolia.rpc.url"
Environment="TOKEN_BRIDGE_CONTRACT=0x6ac02f3887b358591b8b2d22cfb1f36fa5843867"
ExecStart=$(which cosmovisor) run start
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
```
7. Service Activation
```
sudo systemctl daemon-reload
sudo systemctl enable layerd
sudo systemctl start layerd
```
8. Peer and Pruning Settings (thank you ITrocket)
```
sed -i '/^pruning/c\\pruning = "custom"' $HOME/.layer/config/app.toml
sed -i '/^pruning-keep-recent/c\\pruning-keep-recent = "1000"' $HOME/.layer/config/app.toml
sed -i '/^pruning-interval/c\\pruning-interval = "10"' $HOME/.layer/config/app.toml
```
```
PEERS="0e9c659ff5ec9d226b6952d5b42e36daf1b13485@tellor-testnet-peer.itrocket.net:54656,212570ef7791cb48defee09f248cf027d1b987a4@135.181.59.175:54656,472a48ddb5fe43f4687122eef080be6b307f52b8@135.181.79.242:51656,6a1765418a3ea719c80be94d6f4981a02e7b5cb9@152.53.49.146:54656,32ebba9cbd55742244c7bb54dae24afe343f51b1@91.227.33.18:54656,082ee572cd3d9ea0ec8eda650222e015ae546775@185.16.38.165:51656,3cc19186c4a6cec8758b01434a94371a2ba416ec@95.214.55.209:51656,5885d9dc4fe150d8118e308355944274f80865a5@95.214.54.196:51656,a4f1019cc22382af9210f35d30dac5c81199f96f@173.214.171.102:26766,d5519e378247dfb61dfe90652d1fe3e2b3005a5b@213.239.207.162:18456,ed43f4f2aeb0261dc120d25eb1a119ed99b292c4@207.180.197.47:51656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.layer/config/config.toml
```
9.Genesis and Addbook
```
wget -O $HOME/.layer/config/addrbook.json https://server-5.itrocket.net/testnet/tellor/addrbook.json
```
```
wget -O $HOME/.layer/config/genesis.json https://server-5.itrocket.net/testnet/tellor/genesis.json
```
10.If you get a Not command error,
```
cd $HOME/layer && make install
export PATH=$PATH:$HOME/go/bin
```
11.Create Wallet
```
layerd keys add your-wallet-name
```
!!! Don't Forget Save Wallet Key !!!

12.Starter Snap
```
sudo systemctl stop layerd
cp $HOME/.layer/data/priv_validator_state.json $HOME/.layer/priv_validator_state.json.backup
rm -rf $HOME/.layer/data
curl https://server-5.itrocket.net/testnet/tellor/tellor_2025-02-14_439745_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.layer
mv $HOME/.layer/priv_validator_state.json.backup $HOME/.layer/data/priv_validator_state.json
sudo systemctl restart layerd && sudo journalctl -u layerd -f
```
!!! We Cannot Create Validator Without Synchronizing Blocks !!!
13.Create Valıdator.json
```
cd $HOME
echo '{
  "pubkey": {
    "@type": "/cosmos.crypto.ed25519.PubKey",
    "key": "'$(layerd tendermint show-validator | jq -r .key)'"
  },
  "amount": "1000000loya",
  "moniker": "your-monıker-name",
  "commission-rate": "0.1",
  "commission-max-rate": "0.2",
  "commission-max-change-rate": "0.01",
"identity": "optional identity signature (ex. UPort or Keybase)",
	"website": "validator's (optional) website",
	"security": "validator's (optional) security contact email",
	"details": "validator's (optional) details",
  "min-self-delegation": "1"
}' > $HOME/validator.json
```
14. Activate Validator.( You need a test tokens. You want in Discord channel in Tellor)
```
layerd tx staking create-validator $HOME/validator.json --from your-wallet-name --chain-id layertest-3 --fees 10loya --gas auto --gas-adjustment 1.5
```
```
layerd tx staking delegate <VALIDATOR_ADDRESS> 1000000loya --from your-wallet-name --chain-id layertest-3 --fees 10loya --gas auto --gas-adjustment 1.5
```
15.Check From Explorer 
```
https://testnet.dvlnode.com/tellor/staking
```
16.Remove Node and All Files
```
sudo systemctl stop layerd
sudo systemctl disable layerd
sudo rm /etc/systemd/system/layerd.service
sudo rm -rf $HOME/.layer
sudo rm -rf $HOME/layer
sudo rm -rf $HOME/go/bin/layerd $HOME/go/bin/cosmovisor
sudo systemctl daemon-reload
```








