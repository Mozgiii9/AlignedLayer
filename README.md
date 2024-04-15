![image](https://github.com/Mozgiii9/AlignedLayerSetupTheNode/assets/74683169/5faaf46b-854e-476c-9387-c8297d9ebad4)


## Дата создания гайда: 14.04.2024

## Обзор проекта Aligned Layer:

**Aligned Layer** - проект, разработанный компанией Yet Another Company, представляет собой уровень проверки ZK, построенный поверх EigenLayer. Эта инновационная платформа обеспечивает экономичную проверку доказательств SNARK, используя безопасность валидаторов Ethereum и преодолевая ограничения Ethereum. В качестве AVS собственного уровня Aligned Layer предлагает доступные по цене системы проверки и универсальные системы проверки для L2 и мостов, удовлетворяя важнейшие требования в экосистеме блокчейна.

Миссия проекта - повысить безопасность в экосистеме Ethereum. С помощью Aligned Layer расширяются возможности Ethereum в области ZK. "Мы хотим предложить инфраструктуру для будущих приложений, не требующих доверия, с использованием проверяемых вычислений и безопасности Ethereum" - заявили разработчики проекта.

Детали по инвестициям:

**Собрано: 2.600.000$ от таких фондов, как LemniScap, StarkWare, Bankless, Paper Ventures, 01Labs. Проект находится под крылом EigenLayer.**

**Администраторы в Discord'е проекта пишут, что награды за ноду не предусмотрены. Однако, есть множество проектов, которые тоже заявляли о том, что токена не будет, но по итогу выпустили его. Так что делайте собственный ресерч перед тем, как ставить проект. Ниже представлены ссылки на официальные ресурсы проекта:**

- Веб-сайт: [ТЫК](https://alignedlayer.com/);
- Discord: [ТЫК](https://discord.gg/WrgR6YsS);
- Twitter: [ТЫК](https://twitter.com/alignedlayer);
- Github: [ТЫК](https://github.com/yetanotherco/aligned_layer);
- Telegram: [ТЫК](https://t.me/aligned_layer);
- Medium(из РФ доступ только с VPN): [ТЫК](https://blog.yetanothercompany.xyz/aligned-layer/)

## Инструкция по установке ноды:

**1. Установка необходимого ПО:**

```
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential fail2ban ufw
sudo apt -qy upgrade
```

**2. Настройка конфигурации:**

```
MONIKER="<your-moniker-name>"
```

**3. Установка GO:**

```
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.22.0.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

**4. Создание бинарников:**

```
cd $HOME
rm -rf arkeod
wget https://snap.nodex.one/alignedlayer-testnet/alignedlayerd
chmod +x $HOME/alignedlayerd
```

```
mkdir -p $HOME/.alignedlayer/cosmovisor/genesis/bin
mv alignedlayerd $HOME/.alignedlayer/cosmovisor/genesis/bin/
```

```
sudo ln -s $HOME/.alignedlayer/cosmovisor/genesis $HOME/.alignedlayer/cosmovisor/current -f
sudo ln -s $HOME/.alignedlayer/cosmovisor/current/bin/alignedlayerd /usr/local/bin/alignedlayerd -f
```

**5. Установка Cosmos:**

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

**6. Создание сервиса:**

```
sudo tee /etc/systemd/system/alignedlayer.service > /dev/null << EOF
[Unit]
Description=alignedlayer node service
After=network-online.target
 
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.alignedlayer"
Environment="DAEMON_NAME=alignedlayerd"
Environment="UNSAFE_SKIP_BACKUP=true"
 
[Install]
WantedBy=multi-user.target
EOF
```

**7. Включение сервиса:**

```
sudo systemctl daemon-reload
sudo systemctl enable alignedlayer
```

**8. Инициализация ноды:**

```
alignedlayerd init $MONIKER --chain-id alignedlayer
```

```
sed -i \
  -e 's|^chain-id *=.*|chain-id = "alignedlayer"|' \
  -e 's|^keyring-backend *=.*|keyring-backend = "test"|' \
  -e 's|^node *=.*|node = "tcp://localhost:24257"|' \
  $HOME/.alignedlayer/config/client.toml
```

**9. Установка Genesis и Addrbook:**

```
curl -Ls https://snap.nodex.one/alignedlayer-testnet/genesis.json > $HOME/.alignedlayer/config/genesis.json
curl -Ls https://snap.nodex.one/alignedlayer-testnet/addrbook.json > $HOME/.alignedlayer/config/addrbook.json
```

**10. Конфигурация seed'ов:**

```
sed -i -e "s|^seeds *=.*|seeds = \"d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.nodex.one:24210\"|" $HOME/.alignedlayer/config/config.toml
```

```
sed -i -e 's|^persistent_peers *=.*|persistent_peers = "a1a98d9caf27c3363fab07a8e57ee0927d8c7eec@128.140.3.188:26656,1beca410dba8907a61552554b242b4200788201c@91.107.239.79:26656,f9000461b5f535f0c13a543898cc7ac1cd10f945@88.99.174.203:26656,ca2f644f3f47521ff8245f7a5183e9bbb762c09d@116.203.81.174:26656,dc2011a64fc5f888a3e575f84ecb680194307b56@148.251.235.130:20656"|' $HOME/.alignedlayer/config/config.toml
```

**11. Устаналиваем цену газа:**

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0001stake\"|" $HOME/.alignedlayer/config/app.toml
```

**12. Настройка прунинга:**

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.alignedlayer/config/app.toml
```

**13. Установка снэпшотов:**

```
curl -L https://snap.nodex.one/alignedlayer-testnet/alignedlayer-latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.alignedlayer
[[ -f $HOME/.alignedlayer/data/upgrade-info.json ]] && cp $HOME/.alignedlayer/data/upgrade-info.json $HOME/.alignedlayer/cosmovisor/genesis/upgrade-info.json
```

**14. Старт сервиса ноды:**

```
sudo systemctl start alignedlayer
```
