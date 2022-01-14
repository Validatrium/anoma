hardware specs
RAM: minimum 16GB 4
core CPU: 4 core (Intel i3 or better)
Disk: 60GB of free disk space

```bash
# created by Validatrium

# prerequires:
# 1. linux ubuntu
# 2. root account


# install rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# download other dependencies
apt update && apt upgrade -y
apt install -y make git-core libssl-dev pkg-config libclang-12-dev build-essential gcc

# install from source
source $HOME/.cargo/env
git clone https://github.com/anoma/anoma.git
cd anoma
make install
cd ~/

# so that you can run binaries from another user
mv .cargo/bin/* /usr/local/bin/

# some useful variables
echo "export ACCOUNT=<alias>" >> $HOME/.bashrc
# set chain. Testnet id: anoma-testnet-0.0.a1d4bbfafa49
echo "export CHAIN=<chain-id>" >> $HOME/.bashrc
# type `anoma_logs` to get node logs :D 
echo "alias anoma_logs='journalctl -u anoma -f'" >> $HOME/.bashrc
source $HOME/.bashrc

# configuring the node
anoma client utils join-network --chain-id=$CHAIN
# add key
anoma wallet key gen --alias $ACCOUNT # then enter a key password
# create validator account 
anoma client init-account --alias $ACCOUNT --public-key $ACCOUNT --source $ACCOUNT

# The `--base-dir` flag is currently unsupported. So that the way to run the service as an another user
useradd anoma
mkdir /home/anoma
mv -t /home/anoma/ .anoma wasm
chown anoma:anoma /home/anoma /home/anoma/.anoma /home/anoma/wasm -R
ln -s /home/anoma/.anoma /root/

# create service file
[ -f /etc/systemd/system/anoma.service ] || cat > /etc/systemd/system/anoma.service << EOF
[Unit]
Description=anoma
After=network-online.target

[Service]
User=anoma
WorkingDirectory=/home/anoma
ExecStart=anoma ledger  
Restart=always
RestartSec=3
LimitNOFILE=16384
MemoryMax=16G
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
EOF

# start node 
systemctl daemon-reload
systemctl enable anoma
systemctl start anoma
```
Get more info about our projects at our website: Validatrium.com
