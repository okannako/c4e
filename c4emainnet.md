 <h1 align="center">Mainnet Kurulum Rehberi</h1>
 
 ### Tavsiye Edilen Sistem Gereksinimleri
- CPU: 4 cores
- RAM: 16GB RAM 
- SSD: 300GB
- İşletim Sistemi: Ubuntu 20.04LTS

### Kurulum Adımları

## Güncellemeler
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git ncdu -y
sudo apt install make -y && cd $HOME
```

## Go Yüklemesi
```
ver="1.20.3"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## C4e Binary
- İlk 3 kodu sırayla girdikten sonra mutlaka versiyon kontrolü yapın.
```
git clone --depth 1 --branch  v1.3.1  https://github.com/chain4energy/c4e-chain.git
cd c4e-chain/
make install
```
```
c4ed version
output: 1.3.1
```

## C4e Config Ayarları
- Bu adımlarda node'un sıkıntısz çalışması için ağ ve çalışma ayarlarını yapıyoruz. İlk olarak Ağ ve Moniker girişlerimizi yapıyoruz. 
```
export CHAINID=perun-1
export MONIKER=Kendi Moniker Adınız
```

- Inıt İşlemi
```
c4ed init $MONIKER --chain-id $CHAINID
rm -rf ~/.c4e-chain/config/genesis.json
```

- Genesis Taşıma
```
git clone https://github.com/chain4energy/c4e-chains.git
cd c4e-chains/$CHAINID
cp genesis.json ~/.c4e-chain/config/
```

- Config Ayarları
```
peers="5a0e2f1a0541ba34cab61f64b62faaeb4113f2df@65.108.6.45:61056"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.c4e-chain/config/config.toml
```

- Pruning ayarlarını yapmak zorunda değilsiniz. Eğer node'nuz daha az yer kaplasın istiyorsanız aşağıdaki ayaları yapabilirsiniz.
```
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="50" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.c4e-chain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.c4e-chain/config/app.toml
```

- Servis Oluşturma
```
sudo tee /etc/systemd/system/c4ed.service > /dev/null <<EOF
[Unit]
Description=c4e
After=network-online.target

[Service]
User=$USER
ExecStart=$(which c4ed) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

- SnapShot alındığı saate göre nodenuzu ağa en yakın noktadan başlatacaktır, yapmak zorunlu değil. Sisteme daha hızlı bir şekilde eşitlenmenizi sağlar. Burada Obojay'ın yayınlamış olduğu snapshot kullandım.
```
cd $HOME
apt install lz4
sudo systemctl stop c4ed
cp $HOME/.c4e-chain/data/priv_validator_state.json $HOME/.c4e-chain/priv_validator_state.json.backup
rm -rf $HOME/.c4e-chain/data
curl -o - -L http://c4e.snapshot.stavr.tech:1018/c4e/c4e-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.c4e-chain --strip-components 2
mv $HOME/.c4e-chain/priv_validator_state.json.backup $HOME/.c4e-chain/data/priv_validator_state.json
```

- Node Başlatmak
```
sudo systemctl daemon-reload
sudo systemctl enable c4ed
sudo systemctl restart c4ed && sudo journalctl -u c4ed -f -o cat
```

- Cüzdan Oluşturmak
  - Yeni Cüzdan Oluşturmak, Eski Cüzdanı Import Etmek
```
c4ed keys add cuzdanismi
```
```
c4ed keys add cuzdanismi --recover
```

- Node'un sisteme eşit olduğunu bu kodla kontrol ediyoruz. Kodu girdiğinizde false sonucunu veriyorsa node sisteme eşitlenmiştir ve validator oluşturma adımına geçebilirsiniz.
```
c4ed status 2>&1 | jq .SyncInfo
```

- Validator Oluşturma
```
c4ed tx staking create-validator \
  --amount 1000000uc4e \
  --from walletismi \
  --moniker $MONIKER \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(c4ed tendermint show-validator) \
  --chain-id perun-1 \
  --identity="" \
  --details="" \
  --website=""
```



 
