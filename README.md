# Interchain-queries using the new GO v2 Relayer

## 1. Download and build binaries
```
cd
wget https://github.com/Stride-Labs/interchain-queries/archive/refs/tags/v0.0.4.zip
cd interchain-queries-0.0.4
go build
cp /root/interchain-queries-0.0.4/interchain-queries /usr/local/bin
cd
```

## 2. Copy Key From Relayer
```
mkdir -p /root/.icq
cp -r /root/.relayer/keys /root/.icq/keys
```

## 3. create configurations file
```
sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: stride-testnet
chains:
  gaia-testnet:
    key: wallet
    chain-id: GAIA
    rpc-addr: http://<YOUR_GAIA_RPC_ADDR>:<YOUR_GAIA_RPC_PORT>         # Example: '127.0.0.1:23657'
    grpc-addr: http://<YOUR_GAIA_GRPC_ADDR>:<YOUR_GAIA_GRPC_PORT>      # Example: '127.0.0.1:23090'
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001uatom
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  stride-testnet:
    key: wallet
    chain-id: STRIDE-TESTNET-2
    rpc-addr: http://<YOUR_STRIDE_RPC_ADDR>:<YOUR_STRIDE_RPC_PORT>      # Example: '127.0.0.1:16657'
    grpc-addr: http://<YOUR_STRIDE_RPC_ADDR>:<YOUR_STRIDE_GRPC_PORT>    # Example: '127.0.0.1:16090'
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001ustrd
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF
```

## 4. Create icq service
```
tee /etc/systemd/system/interchaind.service > /dev/null <<EOF
[Unit]
Description=InterChainQ
After=network-online.target
[Service]
User=$USER
ExecStart=/usr/local/bin/interchain-queries run --home $HOME/.icq
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## 5. Start icq service
```
systemctl daemon-reload
systemctl enable interchaind
systemctl restart interchaind
```

## 6. Check icq logs
```
journalctl -u interchaind -f -o cat
```

The binary is unusually laconic, so the logs will be "a teaspoon per hour", this is normal, after a while we see:
```
Started InterChainQ.
store/bank/key
Fetching client update for height height 183839
store/bank/key
Fetching client update for height height 183839
Requerying lightblock
Requerying lightblock
Requerying lightblock
Requerying lightblock
Requerying lightblock
Fetching client update for height height 183839
Requerying lightblock
Fetching client update for height height 183839
Send batch of 4 messages
1 ClientUpdate message
1 SubmitResponse message
1 ClientUpdate message
1 SubmitResponse message
Sent batch of 2 (deduplicated) messages
```

After that you can check you transaction in the explorer
