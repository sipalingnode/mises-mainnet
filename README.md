<p style="font-size:14px" align="right">
<a href="https://t.me/autosultan_group" target="_blank">Join our telegram <img src="https://user-images.githubusercontent.com/50621007/183283867-56b4d69f-bc6e-4939-b00a-72aa019d1aea.png" width="30"/></a>
</p>


<p align="center">
  <img height="150" height="auto" src="https://user-images.githubusercontent.com/38981255/204093973-e07f75af-b7e1-4871-83b7-4fd2ad4cdff6.png">
</p>

# TUTORIAL RUNNING NODE MISES MAINNET

## Spek VPS
|  Komponen |  Persyaratan Minimum |
| ------------ | ------------ |
| CPU  | 4 or more physical CPU cores  |
| RAM | At least 4GB of memory (RAM) |
| Penyimpanan  | At least 100GB of SSD disk storage |
| Internet | At least 100mbps network bandwidth |
## Instal Node
```
wget -O mises-mainnet.sh https://raw.githubusercontent.com/sipalingnode/mises-mainnet/main/mises-mainnet.sh && chmod +x mises-mainnet.sh && ./mises-mainnet.sh
```
```
source $HOME/.bash_profile
```
## Buat Wallet
```
misestmd keys add $WALLET
```

## Save Info Wallet
```
MISES_WALLET_ADDRESS=$(misestmd keys show $WALLET -a)
MISES_VALOPER_ADDRESS=$(misestmd keys show $WALLET --bech val -a)
echo 'export MISES_WALLET_ADDRESS='${MISES_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export MISES_VALOPER_ADDRESS='${MISES_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Buat Validator (Membutuhkan setidaknya 2 Mises Mainnet)
**Check status Node dulu kalo udah False boleh lanjut, kalo masih True jangan dulu**
```
misestmd status 2>&1 | jq .SyncInfo
```
```
misestmd tx staking create-validator \
  --amount 1000000umis \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(misestmd tendermint show-validator) \
  --moniker $NODENAME \
  --fees 250umis \
  --chain-id $MISES_CHAIN_ID
```
## Perintah Berguna
**Check Saldo Wallet**
```
misestmd query bank balances $MISES_WALLET_ADDRESS
```
**Check Log Node**
```
journalctl -fu misestmd -o cat
```
**Import Wallet**
```
misestmd keys add $WALLET --recover
```
**Edit Validator**
```
misestmd tx staking edit-validator \
  --moniker="nama-node" \
  --identity="<your_keybase_id>" \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$MISES_CHAIN_ID \
  --fees 250umis \
  --from=$WALLET
```
**Unjail Validator**
```
misestmd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$MISES_CHAIN_ID \
  --fees=250umis
```
**Voting**
```
misestmd tx gov vote 1 yes --from $WALLET --chain-id=$MISES_CHAIN_ID
```
**Delegate**
```
misestmd tx staking delegate $MISES_VALOPER_ADDRESS 1000000umis --from=$WALLET --chain-id=MISES_CHAIN_ID --fees=250umis
```
**Withdraw Reward**
```
misestmd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$MISES_CHAIN_ID --fees=250umis
```
**Withdraw Reward beserta Komisi**
```
misestmd tx distribution withdraw-rewards $MISES_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$MISES_CHAIN_ID
```
**Hapus Node**
```
sudo systemctl stop misestmd && \
sudo systemctl disable misestmd && \
rm /etc/systemd/system/misestmd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf mises-tm && \
rm -rf mises-mainnet.sh && \
rm -rf .misestm && \
rm -rf $(which misestmd)
```
