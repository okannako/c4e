 <h1 align="center">Mainnet Installation Guide</h1>
 
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
sleep 1
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
git clone --depth 1 --branch  v1.0.1  https://github.com/chain4energy/c4e-chain.git
cd c4e-chain/
make install
```
```
c4ed version
output: 1.0.1
```

## C4e Config Ayarları
- Bu adımlarda node'un sıkıntısz çalışması için ağ ve çalışma ayarlarını yapıyoruz. İlk olarak Ağ ve Moniker girişlerimizi yapıyoruz. 
```
export CHAINID=perun-1
export MONIKER=Kendi Moniker Adınız
```
- Inıt İşlemi
```
c4ed init <MONIKER> --chain-id $CHAINID
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
sed -e "s|persistent_peers = \".*\"|persistent_peers = \"$(cat .data | grep -oP 'Persistent peers\s+\K\S+')\"|g" ~/.c4e-chain/config/config.toml > ~/.c4e-chain/config/config.toml.tmp
mv ~/.c4e-chain/config/config.toml.tmp  ~/.c4e-chain/config/config.toml
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

 
