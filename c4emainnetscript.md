 <h1 align="center">Mainnet Tek Kod</h1>
- Aşağıdaki tek kodu çalıştırdıktan sonra sisteme eşitlenmesini beklerken cüzdan ekleme işlemlerini yapabilirsiniz.
```
curl -s https://raw.githubusercontent.com/okannako/c4e/main/c4emainnet.sh > c4emainnet.sh  && chmod +x c4emainnet.sh  && ./c4emainnet.sh 
```

- Cüzdan Ekleme
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
