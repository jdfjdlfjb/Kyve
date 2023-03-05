# Встановлення ноди Kyve у мережі Kaon-1

### Мінімальні вимоги для сервера
## 8GB RAM 200GB of disk space(ssd or nvme) 4Cores CPU

## Оновлюємо наш сервер
```
sudo apt update && sudo apt upgrade -y
```


Встановлюємо додаткові пакети
```
sudo apt install make clang git pkg-config libssl-dev build-essential git gcc chrony curl jq ncdu bsdmainutils htop net-tools lsof fail2ban wget -y
```

Встановлюємо go і перевіряємо версію
```
ver="1.19.4" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
 source $HOME/.bash_profile && \
go version
```

Завантажуємо і переміщуємо бінарник
```
wget https://files.kyve.network/chain/v1.0.0-rc0/kyved_linux_amd64.tar.gz && \
tar -xvzf kyved_linux_amd64.tar.gz  && \
mv kyved /usr/local/bin/ && \
rm kyved_linux_amd64.tar.gz
```

Задаємо змінні (CHAIN залишаємо без змін, в інші вписуємо свої дані)
```
KYVE_CHAIN="kaon-1"
KYVE_MONIKER="your_name"
KYVE_WALLET="your_name"
```

Додаємо все в баш профіль
```
echo 'export KYVE_CHAIN='${KYVE_CHAIN} >> $HOME/.bash_profile 
echo 'export KYVE_MONIKER='${KYVE_MONIKER} >> $HOME/.bash_profile 
echo 'export KYVE_WALLET='${KYVE_WALLET} >> $HOME/.bash_profile 
source $HOME/.bash_profile
```

Ініціалізуємо ноду
```
kyved init $KYVE_MONIKER --chain-id $KYVE_CHAIN
```

Прописуємо в конфіг ім'я мережі
```
kyved  config chain-id $KYVE_CHAIN
```

Скачиваем файл генезис
```
curl https://raw.githubusercontent.com/KYVENetwork/networks/main/kaon-1/genesis.json > ~/.kyve/config/genesis.json
```

Налаштовуємо прунінг (за бажанням)
```
pruning="custom"
pruning_keep_recent="1000"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.kyve/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.kyve/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.kyve/config/app.toml
```

Задаємо мінімальну ціну за газ
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001tkyve\"/;" ~/.kyve/config/app.toml
```

Додаємо піри
```
peers="7258cf2c1867cc5b997baa19ff4a3e13681f14f4@68.183.143.17:26656,e8c9a0f07bc34fb870daaaef0b3da54dbf9c5a3b@15.235.10.35:26656,801fa026c6d9227874eeaeba288eae3b800aad7f@52.29.15.250:26656,bc8b5fbb40a1b82dfba591035cb137278a21c57d@52.59.65.9:26656,430845649afaad0a817bdf36da63b6f93bbd8bd1@3.67.29.225:26656,b68e5131552e40b9ee70427879eb34e146ef20df@18.194.131.3:26656,78d76da232b5a9a5648baa20b7bd95d7c7b9d249@142.93.161.118:26656,97b5c38213e4a845c9a7449b11d811f149fa6710@65.109.85.170:56656,bbb7a427e04d38c74f574f6f0162e1359b66b330@93.115.25.18:39656,1dfe7262db2b9bf51c3b25030e01c89e62640bb1@65.109.71.35:26656,a01d20a3c64a25f5b9199b0273f95cb1471d2b47@65.108.237.231:28656,7820d73c4449e0e4328c9fc4437b00aef8de33c2@5.161.195.113:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.kyve/config/config.toml
```

Створюємо сервісний файл
```
sudo tee /etc/systemd/system/kyved.service > /dev/null <<EOF
[Unit] 
Description=kyve
After=network-online.target
[Service] 
User=$USER
ExecStart=$(which kyved) start --home $HOME/.kyve
Restart=on-failure 
RestartSec=10 
LimitNOFILE=65535
[Install] 
WantedBy=multi-user.target
EOF
```

І запускаємо сервіс
```
sudo systemctl daemon-reload 
sudo systemctl enable kyved
sudo systemctl restart kyved
```

Дивимося логи і чекаємо коли нода почне синхронізуватися
```
sudo journalctl -u kyved -f -o cat
```

Або дивимося статус синхронізації (коли "catching_up": false то нода синхронізована)
```
curl localhost:26657/status
```




