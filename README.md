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
sudo apt install make curl screen tar wget jq build-essential -y 
sudo apt install make clang pkg-config libssl-dev -y
```
## Go kurulumu
```
wget https://go.dev/dl/go1.19.5.linux-amd64.tar.gz
rm -rv /usr/local/go
tar -C /usr/local -xzf go1.19.5.linux-amd64.tar.gz
rm -v go1.19.5.linux-amd64.tar.gz
echo "export PATH= $PATH :/usr/ local/go/bin: $HOME /go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
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
cp $HOME /go/bin/cometbft /usr/local/bin/cometbft
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
## Yedekleme

```
mkdir penumbra_backup && cp validator.json $HOME /penumbra/penumbra_backup/validator_backup.toml
````
## Ağa yükleme

### Gerekli tanımalamalar ve çalıştırma
```
 pcli validator definition template \
    --tendermint-validator-keyfile ~/.penumbra/testnet_data/node0/cometbft/config/priv_validator_key.json \
    --file validator.toml
```
```
cat validator.toml
```
> Aşağıyı aynen yapıştırın. Consensus_key kısmını kendimize göre değiştirelim. üstte çekmiştik. name, website, description kısımlarını değiştirebilirsiniz. Kaydedip çıkalım.
````
# This is a template for a validator definition.
#
# The identity_key and governance_key fields are auto-filled with values derived
# from this wallet's account.
#
# You should fill in the name, website, and description fields.
#
# By default, validators are disabled, and cannot be delegated to. To change
# this, set `enabled = true`.
#
# Every time you upload a new validator config, you'll need to increment the
# `sequence_number`.

sequence_number = 0
enabled = false
name = ''
website = ''
description = ''
identity_key = 'penumbravalid1kqrecmvwcc75rvg9arhl0apsggtuannqphxhlzl34vfamp4ukg9q87ejej'
governance_key = 'penumbragovern1kqrecmvwcc75rvg9arhl0apsggtuannqphxhlzl34vfamp4ukg9qus84v5'

[consensus_key]
type = 'tendermint/PubKeyEd25519'
value = 'HDmm2FmJhLHxaKPnP5Fw3tC1DtlBx8ETgTL35UF+p6w='

[[funding_stream]]
recipient = 'penumbrav2t1cntf73e36y3um4zmqm4j0zar3jyxvyfqxywwg5q6fjxzhe28qttppmcww2kunetdp3q2zywcakwv6tzxdnaa3sqymll2gzq6zqhr5p0v7fnfdaghrr2ru2uw78nkeyt49uf49q'
rate_bps = 100

[[funding_stream]]
recipient = "DAO"
rate_bps = 100
````

```
pcli validator definition upload --file validator.toml
```








