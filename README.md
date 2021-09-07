# Ibc transactions for kichain<->rizon

   In this article I will tell you how to set up a repeater for cross translation between two networks using the example of Rizon and Kichain
Check rizon:
 
 > rizond q ibc-transfer params  # (replace rizond with your node's command)

  Response:
  
 > receive_enabled: true
 > send_enabled: true

 ## Install relayer
 > git clone https://github.com/cosmos/relayer.git && cd relayer

> make install

> cd

> rly version

 Response:

> version: 1.0.0-rc1–152-g112205b

> rly config init
> cd relayer && mkdir rly-config && cd rly-config
> nano kichain-t-4.json

copy the content and place it in a file
{
 "chain-id": "kichain-t-4",
 "rpc-addr": "https://rpc-challenge.blockchain.ki:443", 
 "account-prefix": "tki",
  "gas-adjustment": 1.5,
  "gas-prices": "0.025utki",
  "trusting-period": "48h"
}
 
similarly, we create a file for the second network

> nano groot-011.json

{
 "chain-id": "groot-011",
 "rpc-addr": "http://localhost:26657", #port can be replaced, according to config.toml
 "account-prefix": "rizon",
 "gas-adjustment": 1.5,
 "gas-prices": "0.025uatolo",
 "trusting-period": "48h"
}

Add above both chains to relayer

> rly chains add -f groot-011.json
> rly chains add -f kichain-t-4.json

Create new wallets or restore the ones you already have:
> rly keys add groot-011 name of your wallet 
>rly keys add kichain-t-4 name of your wallet
 
 or we restore:

> rly keys restore groot-011 name of your wallet "mnemonic"
> rly keys restore kichain-t-4 name of your wallet "mnemonic"

Add created keys to the config of the relay

> rly chains edit groot-011 key name of your wallet
> rly chains edit kichain-t-4 key name of your wallet

Change the timeout of waiting for confirmation

> nano ~/.relayer/config/config.yaml

> timeout: 30s
 
Wallets must be funded on both networks,check

> rly q balance groot-011
> rly q balance kichain-t-4

Initialize the light client in both networks

> rly light init groot-011 -f
> rly light init kichain-t-4 -f

Generate a channel between the networks

> rly paths generate groot-011 kichain-t-4 transfer  -- port=transfer

Response:

> Generated path(transfer), run 'rly paths show transfer  -- yaml' to see details

If something went wrong

> rly tx link transfer  --debug

Change the configuration

> nano /root/.relayer/config/config.yaml

Delete sections on both networks

 client-id: ..-tendermint-..
 connection-id: connection-..
 channel-id: channel-..

Re-initialize the light client

> rly light init groot-011 -f
> rly light init kichain-t-4 -f

Run the command to open the channel again
> rly tx link transfer  -- debug

Response:
>  ★ Channel created: [groot-011]chan{channel-11}port{transfer} -> [kichain-t-4]chan{channel-41}port{transfer}

Checking the channel

> rly paths list -d

Response:

> 0: transfer -> chns(✔) clnts(✔) conn(✔) chan(✔) (groot-011:transfer<>kichain-t-3:transfer)

Start cross-network transaction

> rly tx transfer groot-011 kichain-t-4 1000000uatolo tki1...  -- path transfer

Response:

msg(0:transfer) hash(HASH)

Send back

> rly tx transfer kichain-t-4 groot-011 1000000utki rizon1...  -- path transfer

Response:

msg(0:transfer) hash(HASH)

View the address

> rly keys list groot-011
> 
> rly keys list kichaint-t-4
