# ibc-transactions-for-kichain
Hot ro perform IBC transactions in Kichain

This guide describes how to perform IBC transactions between Kichain and Rizon networks. 
We will be setting up the repeater on Rizon node.
Firstly you need to check are IBC-transfers are enabled.

rizond q ibc-transfer params

The output should provide:

receive_enabled: true 
send_enabled: true  

Download and install Relayer v1.0.0 from official source

git clone https://github.com/cosmos/relayer.git
cd relayer
make install
cd
rly version

Correct output is version: 1.0.0-rc1–152-g112205b

Initialize the repeater
rly config init

Make a relayer config directory
mkdir rly_config
cd rly_config

Now we will create a Kichain config file using a global rpc
nano kichain-t-4.json
Insert these settings in the file and save it

{  "chain-id": "kichain-t-4",   "rpc-addr": "https://rpc-challenge.blockchain.ki:443",    "account-prefix": "tki",   "gas-adjustment": 1.5,   "gas-prices": "0.025utki",   "trusting-period": "48h" }  

Also we need to create a file for Rizon chain

nano groot-011.json

Insert these settings in the file and save it

{  "chain-id": "groot-011",  "rpc-addr": "http://localhost:26657",   "account-prefix": "rizon",  "gas-adjustment": 1.5,  "gas-prices": "0.025uatolo",  "trusting-period": "48h" }

Parse config files

rly chains add -f groot-011.json
rly chains add -f kichain-t-4.json
cd

Create wallets for each chain

rly keys add groot-011 <name of your wallet> 
rly keys add kichain-t-4 <ame of your wallet>
  
Parse wallet files
  
rly chains edit groot-011 key name of your wallet
rly chains edit kichain-t-4 key name of your wallet
  
Change standart timeout

nano ~/.relayer/config/config.yaml
timeout: 30s  
  
Next, you will need to get some coins in each wallet. After that, check the availability of funds
  
rly q balance groot-011
rly q balance kichain-t-4
  
Initialize light clients for both networks

rly light init groot-011 -f
rly light init kichain-t-4 -f
  
Generate a path between networks 

rly paths generate groot-011 kichain-t-4 transfer  --port=transfer
  
Open a channel for relaying 
  
rly tx link transfer  --debug
  
If channel doesn't open, try erasing lines containting
client-id: 07-tendermint-16  connection-id: connection-14  channel-id: channel-11

in nano /root/.relayer/config/config.yaml

Re-initialize clients
  
rly light init groot-011 -f
rly light init kichain-t-4 -f

rly tx link transfer  --debug
  
Now by calling this command you will be able to see that we are ready to send transactions

rly paths list -d
  
You can perform transactions with the command

rly transact transfer [src-chain-id] [dst-chain-id] [amount] [dst-addr] [flags] 
  
In my case command should look like following

rly tx transfer groot-011 kichain-t-4 1000000uatolo tki1fza3s64p5uvtusqlqecyxanvlusy9x9h9vqu57 --path transfer

We are transfering tokens from Rizon to Kichain
  
Now we can see our transaction in explorer
https://dev.mintscan.io/rizon/txs/9145F2DC34050FC15635B693A0495548EB4191961527BC428B5F361602406C3C
https://dev.mintscan.io/rizon/txs/1B09C7788AD15E846DBEC18451BEC20FB0E63EC28177EAB093F8B085AD3AF63E

Let's try to send tokens from Kichain to Rizon
https://ki.thecodes.dev/tx/628EB0039EFD8EA9EEB1B560CBF8E3CB76A808243777921410715D5870D7C18E
https://ki.thecodes.dev/tx/289700B0F468F998848FECE98C38889ED238C13799D2ED3EF7F47DB70F9A29B2
  

