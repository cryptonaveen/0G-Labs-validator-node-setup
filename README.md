# **0G labs project overview**

0G is a Modular AI Chain featuring a scalable programmable Data Availability (DA) layer tailored for AI dapps. Its modular technology facilitates seamless interoperability between chains, ensuring security, eliminating fragmentation, and maximizing connectivity.

![320326989-6eca238f-cd35-411b-9c5a-857fbd80dd33](https://github.com/cryptonaveen/0G-Labs-validator-node-setup/assets/116494356/396bb776-ddee-409c-83b1-e6254723fcdb)

## **Hardware requirements**

- Memory: 8 GB RAM
- CPU: 4 cores
- Disk: 500 GB SSD

### **1. Install required packages**

```
sudo apt update
```
```
sudo apt install curl git jq build-essential gcc unzip wget lz4 -y
```

### **2. Install**

```
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### **3. Build evmosd binary**

```
git clone https://github.com/0glabs/0g-evmos.git
```
```
cd 0g-evmos
```
```
git checkout v1.0.0-testnet
```
```
make install
```

         
```
mkdir -p $HOME/.evmosd/cosmovisor/genesis/bin
```
```
mv build/evmosd $HOME/.evmosd/cosmovisor/genesis/bin/
```
```
rm -rf build
```

### **4. system link**

```
sudo ln -s $HOME/.evmosd/cosmovisor/genesis $HOME/.evmosd/cosmovisor/current -f
```

```
sudo ln -s $HOME/.evmosd/cosmovisor/current/bin/evmosd /usr/local/bin/evmosd -f
```

### **5. Cosmovisor download**

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

### **6. Create a service file**

```
sudo tee /etc/systemd/system/ogd.service > /dev/null <<EOF
[Unit]
Description=OG Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which evmosd) start --home $HOME/.evmosd
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### **7. Enable evmos service**

```
sudo systemctl daemon-reload
```

```
sudo systemctl enable evmosd.service
```

### **8. Node settings**

```
evmosd config chain-id zgtendermint_9000-1
```

```
evmosd config keyring-backend os
```

```
evmosd config node tcp://localhost:16457
```

### **9. Initialize the node**  ( replace $MONIKER to your name )

```
evmosd init $MONIKER --chain-id zgtendermint_9000-1
```

Example :  evmosd init Cryptonaveen --chain-id zgtendermint_9000-1

### **10. Download genesis.json**

```
curl -Ls https://github.com/0glabs/0g-evmos/releases/download/v1.0.0-testnet/genesis.json > $HOME/.evmosd/config/genesis.json
```

### **11. Add seeds and peers**

```
PEERS="1248487ea585730cdf5d3c32e0c2a43ad0cda973@peer-zero-gravity-testnet.trusted-point.com:26326" && \
SEEDS="8c01665f88896bca44e8902a30e4278bed08033f@54.241.167.190:26656,b288e8b37f4b0dbd9a03e8ce926cd9c801aacf27@54.176.175.48:26656,8e20e8e88d504e67c7a3a58c2ea31d965aa2a890@54.193.250.204:26656,e50ac888b35175bfd4f999697bdeb5b7b52bfc06@54.215.187.94:26656" && \
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.evmosd/config/config.toml
```

### **12. set minimum gas**

```
sed -i "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00252aevmos\"/" $HOME/.evmosd/config/app.toml
```

### **13. Port settings**

```
echo "export G_PORT="16"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```
sed -i \
    -e "s/\(proxy_app = \"tcp:\/\/\)\([^:]*\):\([0-9]*\).*/\1\2:$PROXY_APP_PORT\"/" \
    -e "s/\(laddr = \"tcp:\/\/\)\([^:]*\):\([0-9]*\).*/\1\2:$RPC_PORT\"/" \
    -e "s/\(pprof_laddr = \"\)\([^:]*\):\([0-9]*\).*/\1localhost:$PPROF_PORT\"/" \
    -e "/\[p2p\]/,/^\[/{s/\(laddr = \"tcp:\/\/\)\([^:]*\):\([0-9]*\).*/\1\2:$P2P_PORT\"/}" \
    -e "/\[p2p\]/,/^\[/{s/\(external_address = \"\)\([^:]*\):\([0-9]*\).*/\1${EXTERNAL_IP}:$P2P_PORT\"/; t; s/\(external_address = \"\).*/\1${EXTERNAL_IP}:$P2P_PORT\"/}" \
    $HOME/.evmosd/config/config.toml
```

```
sed -i \
    -e "/\[api\]/,/^\[/{s/\(address = \"tcp:\/\/\)\([^:]*\):\([0-9]*\)\(\".*\)/\1\2:$API_PORT\4/}" \
    -e "/\[grpc\]/,/^\[/{s/\(address = \"\)\([^:]*\):\([0-9]*\)\(\".*\)/\1\2:$GRPC_PORT\4/}" \
    -e "/\[grpc-web\]/,/^\[/{s/\(address = \"\)\([^:]*\):\([0-9]*\)\(\".*\)/\1\2:$GRPC_WEB_PORT\4/}" $HOME/.evmosd/config/app.toml
```

### **14. Add snap**

```
sudo apt install liblz4-tool
```

```
systemctl stop evmosd
```

```
cp $HOME/.evmosd/data/priv_validator_state.json $HOME/.evmosd/priv_validator_state.json.backup
```

```
cp $HOME/.evmosd/config/priv_validator_key.json $HOME/.evmosd/priv_validator_key.json.backup
```

```
evmosd tendermint unsafe-reset-all --home $HOME/.evmosd --keep-addr-book
```

```
curl -L http://37.120.189.81/0g_testnet/0g_snap.tar.lz4 | tar -I lz4 -xf - -C $HOME/.evmosd
```

```
mv $HOME/.evmosd/priv_validator_state.json.backup $HOME/.evmosd/data/priv_validator_state.json
```

### **15. start the node**

```
sudo systemctl daemon-reload
```

```
sudo systemctl restart evmosd
```

### **16. check the log**

```
sudo journalctl -u evmosd.service -f --no-hostname -o cat
```

### **17. create your validator wallet**  ( replace $WALLET_NAME to your name )

```
evmosd keys add $WALLET_NAME
```

Example :  evmosd keys add Cryptonaveen
Then create a new password and dont forget to save your seed phrase

### **18. Get wallet address**    ( replace $WALLET_NAME to your name )

```
echo "0x$(evmosd debug addr $(evmosd keys show $WALLET_NAME -a) | grep hex | awk '{print $3}')"
```

Then copy your address

### **19. export your private key** ( replace $WALLET_NAME to your name )

```
evmosd keys unsafe-export-eth-key $WALLET_NAME
```

Copy your private key and import on your metamask

### **20. Claim faucet**

- Faucet link [here](https://faucet.0g.ai/)
- 

### **21. Sync your node**

```
evmosd status | jq .SyncInfo.catching_up
```

### **22. Check your balance** ( replace $WALLET_NAME to your name )

```
evmosd q bank balances $(evmosd keys show $WALLET_NAME -a)
```

### **23. create validator**

```
evmosd tx staking create-validator \
  --amount=10000000000000000aevmos \
  --pubkey=$(evmosd tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=$CHAIN_ID \
  --commission-rate=0.05 \
  --commission-max-rate=0.10 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation=1 \
  --from=$WALLET_NAME \
  --identity="" \
  --website="" \
  --details="0G to the moon!" \
  --gas=500000 --gas-prices=99999aevmos \
  -y
```

Note : Change $MONIKER TO your name
           Change $WALLLET_NAME to your wallet name
           you can add identity, you can add your twitter profile link as wabsite

**24. Delegate tokens to your validator**

```
evmosd tx staking delegate $(evmosd keys show $WALLET_NAME --bech val -a)  miktar0000000000000000aevmos --from wallet --gas=500000 --gas-prices=99999aevmos -y
```

Note : NOTE: Type the name of the wallet in the wallet section 2. In the quantity section, write the number 1, if you are going to grain, delete the amount, if it is not 1, reduce it to 1 zero or something.

done, deposit more tokens to your validator and activate your node, stay tuned more updates coming.... 
