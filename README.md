## BitSong 1 to 2 genesis modifier to a single validator

1. `bitsong-1-export.json` - Have been exported by using the command `bitsongd export --height 2482860 --for-zero-height > bitsong-1-export.json`
2. `bitsong-1-migrated.json` - Have been made by using the new `go-bitsong v0.42.x`, by using the command `bitsongd migrate bitsong-1-export.json --chain-id=bitsong-2-test --log_level info > bitsong-1-migrated.json`
3. `genesis.test.json` - The genesis that we are using in out test, have been made by using the script `node migrate.js`

# Install and test

## Update Ubuntu

```
apt update && apt upgrade -y
apt install build-essential git -y
```

## Download and install go

```
wget https://dl.google.com/go/go1.16.8.linux-amd64.tar.gz
tar -xvzf go1.16.8.linux-amd64.tar.gz

cat <<EOF >> ~/.profile
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
```

```
source ~/.profile
```

## Install `go-bitsong`

```
git clone https://github.com/bitsongofficial/go-bitsong.git && cd go-bitsong

git checkout v0.42.x

make install

bitsongd version # 0.8.0-dev-12-g37da72f
```

## Init and start `go-bitsong`

```
bitsongd init <moniker> --chain-id=bitsong-2-test
```

### Optional

```
sed -i 's#"tcp://127.0.0.1:26657"#"tcp://0.0.0.0:26657"#g' ~/.bitsongd/config/config.toml
sed -i 's/enable = false/enable = true/g' ~/.bitsongd/config/app.toml
sed -i 's/swagger = false/swagger = true/g' ~/.bitsongd/config/app.toml
```

### Download the genesis

```
wget https://github.com/bitsongofficial/bitsong-2-test/raw/main/genesis.test.json -O ~/.bitsongd/config/genesis.json
```

### Add peers

```
sed -i 's#persistent_peers = ""#persistent_peers = "795f1c85e983f70d7e4834be37828f9ba036cfdf@94.130.111.95:26656"#g' ~/.bitsongd/config/config.toml
```

### Start

```
bitsongd start --x-crisis-skip-assert-invariants --pruning=nothing
```
