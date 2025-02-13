# Stuck BTC transactions due to too low fee estimation

Date: 2025-02-12

Authors: mar   
Status: DONE

### Summary:
On February 9th, 2025 two users submitted a burn request, to burn an amount of tzBTC using the bridge on get.tzbtc.io. While the Tezos side of their bridge transaction cleared sucessfully and more or less immediately, the transactions on the Bitcoin side did not appear in the Bitcoin mempool.

Investigations have shown that the reason was a fee estimation, which was too low for the transaction to be accepted in the Bitcoin mempool. This happend because the system estimated the fees turing a period of extremely low Bitcoin fees and had no checks or retries in place to ensure to keep fee levels on reasonable levels, so they will be accepted in the mempool.

The two stuck transactions need to be completed manually, by signatures of 3 of the 5 designated keyholders - a setup, which was specifically chosen to recover from such situations. Improvements will be deployed to avoid similar situations in the future.

### Detection: 

The contributing team to the tzBTC bridge was made aware of one stuck transaction (BurnID: 412) early Monday morning, 10 February 2025. The submitting user noticed that no BTC transaction was getting triggered and contacted the team. tzBTC bridge contributors started investigating the root cause. A second user reported the same with his transaction (BurnID: 410).

### Root causes:

The tzBTC bridge system did have an inprecise fee estimation with no lower bound. This caused a Bictoin transaction with a fee that is below the minimum fee most Bitcoin nodes require. 

### Trigger:

During the processing of these two transactions fees on the Bitcoin network were extremely low. This caused the system to forge a Bitcoin transaction with a relay fee that was too small for the transaction to be picked up by (most) Bitcoin nodes. The reason is that most Bitcoin nodes do set a `minrelaytxfee`. If the transaction which was created by the tzBTC bridge carries a lower fee, Bitcoin nodes will reject it from being accepted in the mempool.

### Impact: 

Even though the system tried to broadcast the signed Bitcoin transaction, none of the 5 nodes accepted it into their mempool, which caused the transaction never to appear on the Bitcoin network. 

### Resolution:

tzBTC Bridge contributors were able to get the stuck Bitcoin transactions mined with the help of the AntPool acceleration service / mining pool. All consequent transactions succeeded. A fix was deployed to do a proper estimation and avoid being below the `minrelaytxfee`.

### Action Items:
- stuck transactions were accelerated and were finalized, users got paid
- fix was appllied to avoid the same issue


| Action Item | Type | Owner | State |
| -------- | -------- | -------- |-------- |
| Alert contributors     | mitigate     | mar     |DONE |
| Try to retrigger transaction processing  | mitigate     | luk    |DONE |
| Investigate reasons for failure, system health check | mitigate     | luk,and,ale     |DONE |
| broadcast tranactions with AntPool     | fix     | mar     |DONE |
| develop first fast fix and deploy   | fix     | luk,ale     |DONE |
| inform affected users   | mitigate     | mar     |DONE |
| develop final fix and deploy   | fix     | luk    |ONGOING |


### Timeline:
What was the exact timeline of the relevant events/actions?

(all times CET)

2025-02-10 00:21 User I reports a transaction not being processed via Telegram  
2025-02-10 11:45 User II reports a transaction not being processed via Telegram  
2025-02-10 08:00 tzBTC bridge contributors start investigations  
2025-02-10 17:00 Automated signers are updated  
2025-02-11 16:12 reason is identified, fixes are investigated  
2025-02-11 19:23 a first fix is being deployed, mitigateing most, but not all potential low estimations  
2025-02-12 09:00 a better fix is being developed  
2025-02-12 13:20 AntPool is tasked with tx acceleration  
2025-02-12 14:55 First transaction is mined and succeeds  
2025-02-12 15:44 Second transaction is mined and succeeds  


## Lessons Learned

### What went well
tzBTC contributors worked well together to investigate the root cause. No exploits or misuse potential was open at any time. Other users could still use the bridge without being affected (as their fee estimation was fine).

### What went wrong
The real reason for the transactions failing took a lot of time to be discovered. 

### Where we got lucky
Contributors were able to broadcast the BTC transaction so AntPool was able to mine it, even though most BTC nodes would have rejected it. AntPool included them in the next possible block. System did finalize the transactions immediately.

### Conclusion
Monitoring and error handling should be be improved, so devops can faster find the root cause.
