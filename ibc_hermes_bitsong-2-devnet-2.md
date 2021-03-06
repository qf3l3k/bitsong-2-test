# IBC Relayer Setup - BitSong <-> Sentinel

<!-- MarkdownTOC autolink="true" -->

- [Environment](#environment)
	- [Full nodes](#full-nodes)
	- [Relayer](#relayer)
- [Pre-requisites](#pre-requisites)
	- [Create user](#create-user)
	- [Additional packages](#additional-packages)
- [Relayer installation](#relayer-installation)
	- [Rust installation](#rust-installation)
		- [Relayer deployment](#relayer-deployment)
- [Generating keys](#generating-keys)
	- [Relayer Keys](#relayer-keys)
		- [BitSong](#bitsong)
		- [Sentinel](#sentinel)
	- [Transaction Keys](#transaction-keys)
		- [BitSong \(bitsong-2-devnet-2\)](#bitsong-bitsong-2-devnet-2)
		- [Sentinel \(bluenet-1\)](#sentinel-bluenet-1)
- [Relayer configuration](#relayer-configuration)
	- [Add Keys to Hermes Configuration](#add-keys-to-hermes-configuration)
		- [Sentinel](#sentinel-1)
		- [BitSong](#bitsong-1)
	- [Create channels](#create-channels)
	- [Start Hermes relayer](#start-hermes-relayer)
- [Test transactions](#test-transactions)
	- [BitSong \(bitsong-2-devnet-2\)  Sentinel \(bluenet-1\)](#bitsong-bitsong-2-devnet-2-sentinel-bluenet-1)
		- [From BitSong chain](#from-bitsong-chain)
		- [From Sentinel chain](#from-sentinel-chain)
- [Relayer Management](#relayer-management)
	- [Reload configuration](#reload-configuration)

<!-- /MarkdownTOC -->


# Environment

Operating system: Ubuntu 20.04.3 LTS

## Full nodes
In order to provide environment for relayer 2 full nodes were deployed on same server as Relayer:
 - BitSong (chain-id: bitsong-2-devnet-2)
 - Sentinel (chain-id: bluenet-1)
Installation of full nodes for both chains is not in scope of this guide.

**NOTE:** Nodes were reconfigured in order to co-exist on same server and do not cause port conflicts. Each node is running on dedicated, separate user account for seurity reasons.

## Relayer
Hermes relayer version 0.7.0 will be used
 - Repository: https://github.com/informalsystems/ibc-rs
 - Documentation: https://hermes.informal.systems/


# Pre-requisites

## Create user
```bash
adduser relayer
```
**NOTE:** All Build and configuration steps with Hermes Relayer will be done in context of this user. Running service as normal non-privileged user is good security practice and same time provides separation of the service environment from other services/applications running on server.


## Additional packages
```bash
sudo apt install make clang pkg-config libssl-dev build-essential git jq llvm libudev-dev -y
```

# Relayer installation

**NOTE:** Installation steps should be doe as *hermes* user.

## Rust installation
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```


### Relayer deployment
```bash
git clone https://github.com/informalsystems/ibc-rs.git
cd ibc-rs
git checkout v0.7.0
cargo build --release --bin hermes
```

**NOTE:** Once relayer is built binaries are stores in *${HOME}/ibc-rs/target/release* folder. I moved binaries to *${HOME}/.local/bin* along with binaries for each chain: *kid* and *rizond*. Then logoff and logon, so search path will be added to *${HOME}/.local/bin* folder.


# Generating keys
For testing purposes 4 keys will be creates. Two keys for relayer (one in each chain) and two for transactions (one in each chain).


## Relayer Keys

### BitSong
```bash
bitsongd keys add relayer-bitsong --output json > keys-bitsong.json
```

```json
{
    "name": "relayer-bitsong",
    "type": "local",
    "address": "bitsong1ryczwfsja2wpusfexuaeg34fzwwmfu26tvdc6r",
    "pubkey": "bitsongpub1addwnpepqtc8g6y8s42umqwhhc2uc0s9phzjaflxnprwsk73hhel8kf8zku57cjvd92",
    "mnemonic": "MNEMONIC SEED"
}
```
**NOTE:** Make sure content of JSON file generated by *keys add* has exactly same struture as example above.

### Sentinel
```bash
rizond keys add relayer-sentinel --output json > keys-sentinel.json
Enter keyring passphrase:
Re-enter keyring passphrase:
```

```json
{
    "name": "relayer-sentinel",
    "type": "local",
    "address": "sent1f60g0wjks00zck8nqgq3saa8ffqvtrdhsp97an",
    "pubkey": "sentpub1addwnpepqgrfz5pwjwj4xw43j27fxqcezjrf8sskn209z9m5xeeq7m8rh3wwxvpyyhq",
    "mnemonic": "MNEMONIC SEED"
}
```
**NOTE:** Make sure content of JSON file generated by *keys add* has exactly same struture as example above.



## Transaction Keys
Transaction keys will be used to send test transactions. That **should not** be done using Relayer account.

Keys will be created also on Hermes user account, for testing purposes.
Make sure that binaries (bitsongd and sentinelhub) are available on ${PATH}.


### BitSong (bitsong-2-devnet-2)
```bash
bitsongd keys add qf3l3k-ibc
```

```yaml
- name: qf3l3k-ibc
  type: local
  address: bitsong1w27uqt3m02j82nu0x96ut8yflrj3f0kzlg5lgc
  pubkey: bitsongpub1addwnpepqwh2x5rrgjunq6radzze9v62uykxgu82se53scuc7vvpq0lk5wx36nq4ajx
  mnemonic: ""
  threshold: 0
  pubkeys: []


**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

MNEMONIC SEED
```

### Sentinel (bluenet-1)
```bash
sentinelhub keys add qf3l3k-ibc
```

```yaml
Enter keyring passphrase:

- name: qf3l3k-ibc
  type: local
  address: sent19s8flsrd0u93c6x94ln49sl0fuxnp92j4awxrk
  pubkey: sentpub1addwnpepq24q9hgvxutjyt4grmcmrk3039yhczcz5q4ntxzs9e89av33sy2fjq7n72x
  mnemonic: ""
  threshold: 0
  pubkeys: []


**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

MNEMONIC SEED
```


# Relayer configuration

**All activity will be done in context of *hermes* user, which is account created specifically to run relayer.**

Default folder for Hermes is *${HOME}/.hermes*, so it needs ot be created
```bash
mkdir -p ${HOME}/.hermes
```

Configuration file *${HOME}/.hermes/config.toml*
```toml
[global]
strategy = 'packets'
log_level = 'info'

[telemetry]
enabled = true
host    = '127.0.0.1'
port    = 3001

[rest]
enabled = true
host    = '127.0.0.1'
port    = 3000

[[chains]]
id = 'bitsong-2-devnet-2'
rpc_addr = 'http://127.0.0.1:26677'
grpc_addr = 'http://127.0.0.1:9110'
websocket_addr = 'ws://localhost:26677/websocket'
rpc_timeout = '10s'
account_prefix = 'bitsong'
key_name = 'relay-bitsong'
store_prefix = 'ibc'
max_gas = 2000000
gas_price = { price = 0.001, denom = 'ubtsg' }
gas_adjustment = 0.1
clock_drift = '5s'
trusting_period = '14days'
trust_threshold = { numerator = '1', denominator = '3' }

[[chains]]                                              
id = 'bluenet-1'                                        
rpc_addr = 'http://127.0.0.1:26697'                     
grpc_addr = 'http://127.0.0.1:9130'                     
websocket_addr = 'ws://127.0.0.1:26697/websocket'       
rpc_timeout = '15s'                                     
account_prefix = 'sent'                                 
key_name = 'relay-sent'                                 
store_prefix = 'ibc'                                    
max_gas = 2000000                                       
gas_price = { price = 0.25, denom = 'ublue' }           
gas_adjustment = 0.1                                    
clock_drift = '10s'                                     
trusting_period = '2days'                               
trust_threshold = { numerator = '1', denominator = '3' }
```

**NOTE:** Ports for RPC services were changed as both nodes are running on same machine. THis is specific to test configuration.


## Add Keys to Hermes Configuration

### Sentinel
```bash
hermes keys add bluenet-1 -f keys-sentinel.json

Sep 02 22:46:03.939  INFO using default configuration from '/home/relayer/.hermes/config.toml'
Success: Added key 'relay-sent' (sent1zuv7qy786msa86npvpuze876wukg2zu5c2s0cs) on chain sentinelhub-2
```

### BitSong
```bash
hermes keys add bitsong-2-test -f keys-bitsong.json -p "m/44'/639'/0'/0/0"

Sep 21 22:57:49.424  INFO using default configuration from '/home/relayer/.hermes/config.toml'
Success: Added key 'relay-bitsong' (bitsong1ryczwfsja2wpusfexuaeg34fzwwmfu26tvdc6r) on chain bitsong-2-test
```


## Create channels
```bash
hermes create channel bitsong-2-devnet-2 bluenet-1 --port-a transfer --port-b transfer
```


Below example results of channel creation
```bash
Sep 22 19:13:19.720  INFO using default configuration from '/home/relayer/.hermes/config.toml'
Sep 22 19:13:19.781  WARN Hermes might be misconfigured for chain 'bitsong-2-devnet-2'
Sep 22 19:13:19.781  WARN     Reason:
   0: semantic config validation failed for option `max_tx_size` chain 'bitsong-2-devnet-2', reason: `max_tx_size` = 2097152 is greater than 90% of the genesis block param `max_size` = 2000
00

Location:
   /home/relayer/.cargo/registry/src/github.com-1ecc6299db9ec823/flex-error-0.4.2/src/tracer_impl/eyre.rs:10

Backtrace omitted.
Run with RUST_BACKTRACE=1 environment variable to display it.
Run with RUST_BACKTRACE=full to include source snippets.
Sep 22 19:13:19.781  WARN     Some Hermes features may not work in this mode!
Sep 22 19:13:19.789  INFO Creating new clients, new connection, and a new channel with order ORDER_UNORDERED
Sep 22 19:13:19.797  WARN [bitsong-2-devnet-2] waiting for commit of tx hashes(s) 0CD18BA8F69C397891DC4EA185F389DCB837E5D20DB8F6C2092C5C87310551F3
Sep 22 19:13:23.605  INFO ???? [bluenet-1 -> bitsong-2-devnet-2:07-tendermint-4]  => CreateClient(
    CreateClient(
        Attributes {
            height: Height {
                revision: 2,
                height: 13610,
            },
            client_id: ClientId(
                "07-tendermint-4",
            ),
            client_type: Tendermint,
            consensus_height: Height {
                revision: 1,
                height: 177422,
            },
        },
    ),
)

Sep 22 19:13:23.614  WARN [bluenet-1] waiting for commit of tx hashes(s) 2A811ED936828A471148D78CD7C76672D936D3857FC20E39B7C3A25A95C85719
Sep 22 19:13:27.122  INFO ???? [bitsong-2-devnet-2 -> bluenet-1:07-tendermint-3]  => CreateClient(
    CreateClient(
        Attributes {
            height: Height {
                revision: 1,
                height: 177424,
            },
            client_id: ClientId(
                "07-tendermint-3",
            ),
            client_type: Tendermint,
            consensus_height: Height {
                revision: 2,
                height: 13610,
            },
            },
        },
    ),
)

Sep 22 19:13:27.125  WARN [bitsong-2-devnet-2] waiting for commit of tx hashes(s) 425C0928C9AABEF6F27A00A81E52CCCC3F556966B077CE9D45FD297AF72D74B9
????  bitsong-2-devnet-2 => OpenInitConnection(
    OpenInit(
        Attributes {
            height: Height {
                revision: 2,
                height: 13611,
            },
            connection_id: Some(
                ConnectionId(
                    "connection-4",
                ),
            ),
            client_id: ClientId(
                "07-tendermint-4",
            ),
            counterparty_connection_id: None,
            counterparty_client_id: ClientId(
                "07-tendermint-3",
            ),
        },
    ),
)

Sep 22 19:13:28.247  WARN [bitsong-2-devnet-2] waiting for commit of tx hashes(s) E9B0A6453F7D77F41C2BA4DFDA71DE4AEAAC1E297FCE026A9364B1FBFD2E210F
Sep 22 19:13:38.012  WARN [bluenet-1] waiting for commit of tx hashes(s) BA75824FB212B04B4B570E266D27B836CB01001AFB34E85AB6F71A181BB7AF41
????  bluenet-1 => OpenTryConnection(
    OpenTry(
        Attributes {
            height: Height {
                revision: 1,
                height: 177427,
            },
            connection_id: Some(
                ConnectionId(
                    "connection-2",
                ),
            ),
            client_id: ClientId(
                "07-tendermint-3",
            ),
            counterparty_connection_id: Some(
                ConnectionId(
                    "connection-4",
                ),
            ),
            counterparty_client_id: ClientId(
                "07-tendermint-4",
            ),
        },
    ),
)

Sep 22 19:13:43.640  WARN [bluenet-1] waiting for commit of tx hashes(s) 601E53A757D4E6F0F6E7A984DA0061331986881D35339B05ECA05B179E09C2E2
Sep 22 19:13:54.314  WARN [bitsong-2-devnet-2] waiting for commit of tx hashes(s) F7C19ECEC453976EAAA584035C43D294CEC44273C42C0FA457C7282B69E1A53D
????  bitsong-2-devnet-2 => OpenAckConnection(
    OpenAck(
        Attributes {
            height: Height {
                revision: 2,
                height: 13617,
            },
            connection_id: Some(
                ConnectionId(
                    "connection-4",
                ),
            ),
            client_id: ClientId(
                "07-tendermint-4",
            ),
            counterparty_connection_id: Some(
                ConnectionId(
                    "connection-2",
                ),
            ),
            counterparty_client_id: ClientId(
                "07-tendermint-3",
            ),
        },
    ),
)

Sep 22 19:14:02.872  WARN [bluenet-1] waiting for commit of tx hashes(s) 78772C1309C63737782A51BA04F9B822503B6F2193D28BAC20CB67D49CB68CCA
????  bluenet-1 => OpenConfirmConnection(
    OpenConfirm(
        Attributes {
            height: Height {
                revision: 1,
                height: 177431,
            },
            connection_id: Some(
                ConnectionId(
                    "connection-2",
                ),
            ),
            client_id: ClientId(
                "07-tendermint-3",
            ),
            counterparty_connection_id: Some(
                ConnectionId(
                    "connection-4",
                ),
            ),
            counterparty_client_id: ClientId(
                "07-tendermint-4",
            ),
        },
    ),
)

????????????  Connection handshake finished for [Connection {
    delay_period: 0ns,
    a_side: ConnectionSide {
        chain: ProdChainHandle {
            chain_id: ChainId {
                id: "bitsong-2-devnet-2",
                version: 2,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-4",
        ),
        connection_id: Some(
            ConnectionId(
                "connection-4",
            ),
        ),
    },
    b_side: ConnectionSide {
        chain: ProdChainHandle {
            chain_id: ChainId {
                id: "bluenet-1",
                version: 1,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-3",
        ),
        connection_id: Some(
            ConnectionId(
                "connection-2",
            ),
        ),
    },
}]

Sep 22 19:14:05.784  WARN [bitsong-2-devnet-2] waiting for commit of tx hashes(s) 72499A45044430D29770F33A1E270B713E58F07BB7F2D374DE0A95AFDEEBFB58
Sep 22 19:14:08.389  INFO done bitsong-2-devnet-2 => OpenInitChannel(
    OpenInit(
        Attributes {
            height: Height {
                revision: 2,
                height: 13619,
            },
            port_id: PortId(
                "transfer",
            ),
            channel_id: Some(
                ChannelId(
                    "channel-4",
                ),
            ),
            connection_id: ConnectionId(
                "connection-4",
            ),
            counterparty_port_id: PortId(
                "transfer",
            ),
            counterparty_channel_id: None,
        },
    ),
)

Sep 22 19:14:08.389  INFO successfully opened init channel
Sep 22 19:14:12.940  WARN [bluenet-1] waiting for commit of tx hashes(s) 273838813D9CB4CE7301DDEB16ABA1394195F4FE1E51C0DD8D8BC27D0FE49D31
done bluenet-1 => OpenTryChannel(
    OpenTry(
        Attributes {
            height: Height {
                revision: 1,
                height: 177433,
            },
            port_id: PortId(
                "transfer",
            ),
            channel_id: Some(
                ChannelId(
                    "channel-2",
                ),
            ),
            connection_id: ConnectionId(
                "connection-2",
            ),
            counterparty_port_id: PortId(
                "transfer",
            ),
            counterparty_channel_id: Some(
                ChannelId(
                    "channel-4",
                ),
            ),
        },
    ),
)

Sep 22 19:14:21.811  WARN [bitsong-2-devnet-2] waiting for commit of tx hashes(s) C90AD63C47ABF04BFEF14A6372C8A7F26DC6F5D4A727B8BEE2F7F31154968866
Sep 22 19:14:23.816  INFO done with ChanAck step bitsong-2-devnet-2 => OpenAckChannel(
    OpenAck(
        Attributes {
            height: Height {
                revision: 2,
                height: 13622,
            },
            port_id: PortId(
                "transfer",
            ),
            channel_id: Some(
                ChannelId(
                    "channel-4",
                ),
            ),
            connection_id: ConnectionId(
                "connection-4",
            ),
            counterparty_port_id: PortId(
                "transfer",
            ),
            counterparty_channel_id: Some(
                ChannelId(
                    "channel-2",
                ),
            ),
        },
    ),
)

Sep 22 19:14:28.268  WARN [bluenet-1] waiting for commit of tx hashes(s) 24D17E5264C3A1C37B24A98A9D8EEB0C602A8E28943EBDC6E6599F526E145CE4
Sep 22 19:14:33.282  INFO done bluenet-1 => OpenConfirmChannel(
    OpenConfirm(
        Attributes {
            height: Height {
                revision: 1,
                height: 177436,
            },
            port_id: PortId(
                "transfer",
            ),
            channel_id: Some(
                ChannelId(
                    "channel-2",
                ),
            ),
            connection_id: ConnectionId(
                "connection-2",
            ),
            counterparty_port_id: PortId(
                "transfer",
            ),
            counterparty_channel_id: Some(
                ChannelId(
                    "channel-4",
                ),
            ),
        },
    ),
)

Success: Channel {
    ordering: Unordered,
    a_side: ChannelSide {
        chain: ProdChainHandle {
            chain_id: ChainId {
                id: "bitsong-2-devnet-2",
                version: 2,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-4",
        ),
        connection_id: ConnectionId(
            "connection-4",
        ),
        port_id: PortId(
            "transfer",
        ),
        channel_id: Some(
            ChannelId(
                "channel-4",
            ),
        ),
    },
    b_side: ChannelSide {
        chain: ProdChainHandle {
            chain_id: ChainId {
                id: "bluenet-1",
                version: 1,
            },
            runtime_sender: Sender { .. },
        },
        client_id: ClientId(
            "07-tendermint-3",
        ),
        connection_id: ConnectionId(
            "connection-2",
        ),
        port_id: PortId(
            "transfer",
        ),
        channel_id: Some(
            ChannelId(
                "channel-2",
            ),
        ),
    },
    connection_delay: 0ns,
    version: Some(
        "ics20-1",
    ),
}                                                                            
```


## Start Hermes relayer
```bash
hermes start
```


# Test transactions

## BitSong (bitsong-2-devnet-2) <-> Sentinel (bluenet-1)

### From BitSong chain
```bash
bitsongd tx ibc-transfer transfer transfer channel-4 sent19s8flsrd0u93c6x94ln49sl0fuxnp92j4awxrk 111899ubtsg --node http://localhost:26677 --from qf3l3k-ibc --chain-id bitsong-2-devnet-2
```

Check balance of the destination account

```bash
sentinelhub query bank balances sent19s8flsrd0u93c6x94ln49sl0fuxnp92j4awxrk --node http://localhost:26697
balances:
- amount: "223798"
  denom: ibc/3390C9DF6058E28678A565FC39A50D5B2ACACBBCA9505E073D4052882AC9B563
- amount: "111899"
  denom: ibc/FE393788C41E65BEB325ECA8A42FDB5C1676E7BDAC8AB5DB04E34547AC53B262
- amount: "989464303"
  denom: ublue
pagination:
  next_key: null
  total: "0"
```

### From Sentinel chain
```bash
sentinelhub tx ibc-transfer transfer transfer channel-2 bitsong1w27uqt3m02j82nu0x96ut8yflrj3f0kzlg5lgc 111899ublue --node http://localhost:26697 --from qf3l3k-ibc --chain-id bluenet-1 --gas auto --fees 2000000ublue
```


Check balance of the destination account
```bash
bitsongd query bank balances bitsong1w27uqt3m02j82nu0x96ut8yflrj3f0kzlg5lgc --node http://localhost:26677
balances:
- amount: "223798"
  denom: ibc/A96C994A9DB01B82FDC9AF63313569BAF96424F301E5E9096E32FDC0D81FA138
- amount: "1776202"
  denom: ubtsg
pagination:
  next_key: null
  total: "0"
```


# Relayer Management

## Reload configuration
```bash
ps aux | rg hermes | awk '{ print $2 }' | head -n1 | xargs -I{} kill -SIGHUP {}
```
