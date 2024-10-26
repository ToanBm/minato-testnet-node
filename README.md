# Minato Node Setup
# Binary installation
## Prerequisites
- Ubuntu 22.04 or a compatible Linux distribution.
- Root or sudo privileges.
- OpenSSL installed.
## Installation Steps
### Step 1: Download Binaries
Download the `op-node` and `geth` binaries from the release page:
```bash
wget https://github.com/Soneium/soneium-node/releases/download/1.9.3-e81c50de-1727294020/op-node
wget https://github.com/Soneium/soneium-node/releases/download/1.101408.0-stable-5c2e7586/geth
```
### Step 2: Set Executable Permissions
Make the downloaded binaries executable:
```bash
chmod +x op-node geth
```
### Step 3: Move Binaries to `/usr/local/bin`
Move the binaries to `/usr/local/bin` for easy execution:
```bash
sudo mv -t /usr/local/bin geth op-node
```
### Step 4: Set Up Configuration
Create the necessary directories and generate the JWT secret:
```bash
sudo mkdir /etc/optimism
git clone https://github.com/Soneium/soneium-node.git
cd soneium-node/minato
openssl rand -hex 32 > jwt.txt
sudo mv -t /etc/optimism/ minato-genesis.json jwt.txt minato-rollup.json
```
### Step 5: Initialize Geth
Initialize `geth` with the genesis configuration:
```bash
sudo geth init --state.scheme=hash --datadir=/data/optimism/ /etc/optimism/minato-genesis.json
```
## Service Configuration
<b>`op-node` Service</b>
Create a systemd service for op-node:
```bash
sudo nano /etc/systemd/system/op-node.service
```
Add the following content:
- Change your info:
```bash
[Unit]
Description=op-node
After=network-online.target
Wants=network-online.target

[Service]
User=root
Group=root
Type=simple

ExecStart=/usr/local/bin/op-node \
  --l1=https://sepolia-l1.url \
  --l2=http://localhost:8551 \
  --rpc.addr=127.0.0.1 \
  --rpc.port=19545 \
  --l2.jwt-secret=/etc/optimism/jwt.txt \
  --l1.trustrpc \
  --l1.beacon=https://ethereum-sepolia-beacon-api.publicnode.com \
  --syncmode=execution-layer \
  --l1.rpckind=erigon \
  --p2p.priv.path=/etc/optimism/p2p.key \
  --p2p.static=/dns4/peering-01.prd.hypersonicl2.com/tcp/19222/p2p/16Uiu2HAm36ufaFmS3tjSjkUnwSJmQN8W8fZ8yXiu2AYL2o11EgcK,/dns4/peering-02.prd.hypersonicl2.com/tcp/19222/p2p/16Uiu2HAmPkRbG8kkhJ3JWmrqeiMvy1hWXFSz4s4rncVe8YiCJHmx \
  --p2p.discovery.path=/etc/optimism/p2p.db \
  --p2p.peerstore.path=/etc/optimism/p2p-peerstore.db \
  --metrics.enabled \
  --p2p.advertise.ip=<your-public-ip> \
  --p2p.listen.tcp=19222 \
  --metrics.port=7310 \
  --override.fjord=1730106000 \
  --override.granite=1730106000 \
  --rollup.config=/etc/optimism/minato-rollup.json

Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
<b>`op-geth` Service</b>
Create a systemd service for `geth`:
```bash
sudo nano /etc/systemd/system/op-geth.service
```
Add the following content:
- Change your info:
```bash
[Unit]
Description=op-geth
After=network-online.target
Wants=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/bin/geth \
  --datadir=/data/optimism \
  --http \
  --http.port=18545 \
  --http.corsdomain=* \
  --http.vhosts=* \
  --http.addr=0.0.0.0 \
  --http.api=web3,debug,eth,txpool,net,engine \
  --ws \
  --ws.addr=0.0.0.0 \
  --ws.port=18546 \
  --ws.origins=* \
  --ws.api=debug,eth,txpool,net,engine \
  --syncmode=full \
  --gcmode=archive \
  --maxpeers=100 \
  --authrpc.vhosts=* \
  --authrpc.addr=0.0.0.0 \
  --authrpc.port=8551 \
  --authrpc.jwtsecret=/etc/optimism/jwt.txt \
  --metrics \
  --metrics.addr=0.0.0.0 \
  --metrics.expensive \
  --metrics.port=6060 \
  --rollup.disabletxpoolgossip=false \
  --rpc.allow-unprotected-txs=true \
  --nat=extip:<your-public-ip> \
  --db.engine=pebble \
  --state.scheme=hash \
  --override.fjord=1730106000 \
  --override.granite=1730106000 \
  --port=40303
  --bootnodes=enode://6526c348274c54e7b4184014741897eb25e12ca388f588b0265bb2246caeea87ed5fcb2d55b7b08a90cd271a53bc76decb6d1ec37f219dbe4cd3ed53a888118b@peering-02.prd.hypersonicl2.com:40303,enode://34f172c255b11f64828d73c90a60395691e89782639423d434385594dd38b434ddffb78ad411da6fd37cbda6d0f93e17ceae399ac4f2594b0d54eb8c83c27de9@peering-01.prd.hypersonicl2.com:40303


Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
## Step 6: Enable and Start Services
Enable and start the services to run at boot:
```bash
sudo systemctl enable op-node.service
sudo systemctl start op-node.service
```
```bash
sudo systemctl enable op-geth.service
sudo systemctl start op-geth.service
```
Check logs:
```bash
sudo journalctl -u op-geth.service -f
sudo journalctl -u op-node.service -f
```
## Stop and restart node
```bash
sudo systemctl stop op-node.service
sudo systemctl stop op-geth.service
```
```bash
sudo systemctl daemon-reload
sudo systemctl restart op-node
sudo systemctl restart op-geth
```
## Show your key
```bash
sudo cat /etc/optimism/jwt.txt
```


