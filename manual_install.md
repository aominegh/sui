<p style="font-size:14px" align="right">
<a href="https://discord.gg/hcbT5ryUdF" target="_blank">Join our discord <img src="[https://user-images.githubusercontent.com/50621007/168689534-796f181e-3e4c-43a5-8183-9888fc92cfa7.png](https://camo.githubusercontent.com/0ef309f7e0b554033dd25b3ce83015db2f0f8952fb4c31318af095369d3d4453/68747470733a2f2f7669676e657474652e77696b69612e6e6f636f6f6b69652e6e65742f7468652d6d696e6572732d686176656e2d70726f6a6563742f696d616765732f642f64642f446973636f72642e706e672f7265766973696f6e2f6c61746573743f63623d3230313730333038303333353436)" width="30"/></a>
</p>

<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/174559198-c1f612e5-bba2-4817-95a8-8a3c3659a2aa.png">
</p>

# Install Sui node
To setup Sui node follow the steps below

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install tzdata git ca-certificates curl build-essential libssl-dev pkg-config libclang-dev cmake jq -y --no-install-recommends
```

## Install yq
```
sudo wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/download/v4.23.1/yq_linux_amd64 && sudo chmod +x /usr/local/bin/yq
```

## Install Rust
```
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
source $HOME/.cargo/env
```

## Download and build Sui binaries
```
sudo mkdir -p /var/sui
cd $HOME && rm sui -rf
git clone https://github.com/MystenLabs/sui.git && cd sui
git remote add upstream https://github.com/MystenLabs/sui
git fetch upstream
git checkout --track upstream/devnet
cargo build --release -p sui-node
sudo mv ~/sui/target/release/sui-node /usr/local/bin/
```

## Set configuration
```
wget -O /var/sui/genesis.blob https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob
sudo cp crates/sui-config/data/fullnode-template.yaml /var/sui/fullnode.yaml
sudo yq e -i '.db-path="/var/sui/db"' /var/sui/fullnode.yaml \
&& yq e -i '.genesis.genesis-file-location="/var/sui/genesis.blob"' /var/sui/fullnode.yaml \
&& yq e -i '.metrics-address="0.0.0.0:9184"' /var/sui/fullnode.yaml \
&& yq e -i '.json-rpc-address="0.0.0.0:9000"' /var/sui/fullnode.yaml
```

## Create and run service
```
echo "[Unit]
Description=Sui Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/sui-node --config-path /var/sui/fullnode.yaml
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/suid.service
mv $HOME/suid.service /etc/systemd/system/

sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF

sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable suid
sudo systemctl restart suid
```
