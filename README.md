# Penumbra node kurulumu
## Sistem gereksinimleri
 - Ubuntu 22.04+
 - 4-8 GB RAM
 - 2-4 vCPUs
 - 200 GB storage
 - Portlar : 26656,26657,26658,8080,443,9000
 - [Resmi kaynak](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)

## Sistem günceleme ve gerekli paketlerin kurulumu

```
sudo apt update && sudo apt upgrade -y
sudo apt install git make curl screen tar wget jq build-essential -y 
sudo apt install make clang pkg-config libssl-dev -y
```
## Go kurulumu
```
wget -q -O - https://go.dev/dl/go1.19.5.linux-amd64.tar.gz | sudo tar xvzf - -C /usr/local
```
```
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```


## Pd kurulumu

```
curl -O -L https://github.com/penumbra-zone/penumbra/releases/download/v0.64.1/pd-x86_64-unknown-linux-gnu.tar.xz
unxz pd-x86_64-unknown-linux-gnu.tar.xz
tar -xf pd-x86_64-unknown-linux-gnu.tar
sudo mv pd-x86_64-unknown-linux-gnu/pd /usr/local/bin/
```
- Çalıştığını kontrol edelim.
```
pd --version
```
## Cometbft kurulumu.

```
cd $HOME 
git clone https://github.com/cometbft/cometbft.git 
cd cometbft 
git checkout v0.37.2 
make install
make build
cp $HOME/cometbft/build/cometbft /usr/local/bin/cometbft
````
## Reset data ve testnete katılma

```
pd testnet unsafe-reset-all
pd testnet join
```

> Screen açalım ve tendermint çalıştıralım
```
screen -S cometbft
cometbft start --home ~/.penumbra/testnet_data/node0/cometbft
```
> Burdan ctrl+a+d ile çıkalım.

## Pd çalıştıralım

```
screen -S pd
pd start
```
## Konsensus key çekme (not defterine kopyalayın)
```
grep -A3 pub_key ~/.penumbra/testnet_data/node0/cometbft/config/priv_validator_key.json
```

## Ağa yükleme

### Gerekli tanımalamalar ve çalıştırma
```
 pcli validator definition template \
    --tendermint-validator-keyfile ~/.penumbra/testnet_data/node0/cometbft/config/priv_validator_key.json \
    --file validator.toml
```
```
nano validator.toml
```
> Name kısmını değiştirelim.Diğerlerini kontrol edelim.


```
pcli validator definition upload --file validator.toml
```








