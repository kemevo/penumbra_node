# Telegram
- https://t.me/HerculesNode
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
# Seremoni kısmı hariç, önce [burayı](https://github.com/kemevo/Penumbra-Seremoni) pcli yükleme kısmından itibaren yapın. Seremoniyi sonra yaparsınız.
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
curl -O -L https://github.com/penumbra-zone/penumbra/releases/download/v0.64.2/pd-x86_64-unknown-linux-gnu.tar.xz
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
> Burdan ctrl+a+d ile çıkalım.
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
> Name kısmını,false olan yeri true yapalım. değiştirelim.Diğerlerini kontrol edelim.

## Başlatma
```
pcli validator definition upload --file validator.toml
```
## Delegate
> Aşağıdaki komutun çıktısını kopyalayalım.
```
pcli validator identity
````
```
pcli tx delegate 1penumbra --to **üstteki çıktı buraya gelecek**
```
> Validator listesi
```
pcli query validator list -i
```
> Bir süre sonra burda isminiz gözükecek

## BAZI KOMUTLAR
> Token gönderme

```
pcli tx send 10penumbra --to "GIDECEK_ADRES"
```
> Delegate & Undelegate
```
pcli tx delegate 10penumbra --to "VALIDATOR_ADRES"
```
```
pcli tx undelegate-claim
```
> Likitide açma,kapatma,swap 
```
pcli tx position order buy 1gm@1penumbra
#1 gm yi 1 penumbra fiyattan fee olmadan almak için
```
```
pcli tx position order sell 1penumbra@1gm/20bps
#1 penumbra yi 1 gm fiyattan 20 basepoint fee ödeyerek satmak için
```
```
pcli view balance
# Açık emirler gözükür
 Account  Amount
 0        1lpnft_opened_plpid1hzrzr2myjw508nf0hyzehl0w0x2xzr4t8vwe6t3qtnfhsqzf5lzsufscqr
```
```
pcli tx position close plpid1hzrzr2myjw508nf0hyzehl0w0x2xzr4t8vwe6t3qtnfhsqzf5lzsufscqr
# Açık emri kapatmak için
pcli tx position close-all
# Hepsini kapatmak için
```
```
pcli tx swap --into gm 1penumbra
# gm token alır 1 penumbra karşılığında mevcut likitide emirlerinden
```
> Oylama
```
pcli query governance list-proposals
# mevcut oylamalar listelenir
```
```
pcli tx vote yes --on 1
# Delegator olarak oylama 1 için evet oyu kullanır.
```
```
ppcli validator vote yes --on 1
# Validator(aktif) olarak oylama 1 için evet oyu kullanır.
```









