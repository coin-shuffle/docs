# Overview

For further explanation, we will use the following terms:

+ **Participant** - a user that wants to shuffle his UTXOs.
+ **Input** - a UTXO that participant wants to shuffle, spent.
+ **Output** - an address that will receive a UTXO after shuffling.
+ **Encrypted Outputs** - a list of **outputs** encrypted by RSA secret key. 
+ **Service** - a service that provides **coordination** logic for participants
    without any ability to reveal **output** addresses or steal **UTXO**s.
+ **Room** - a group of participants that are currently shuffling their **UTXO**s.

The shuffling process is done in the following steps:

1. **Participants Registration**. Participants register in the **service** by providing
    their proof of ownership of **UTXO**s that they want to shuffle.
2. **Participants connection**. Participants send their RSA public keys to the
   **service** to notify service that they are ready for shuffling.
3. **Keys distribution** - **service** defines order of shuffling and sends RSA
   public keys that are required for **outputs** **encryption** and
   **decryption** to each participant.
4. **Shuffling**. **Service** sends **encrypted outputs** to each participant,
   participants partially **decrypting** them (_with their own keys_),
   **encrypting** their **output** and sending them to **service** which will
   pass them to next participant.
5. **Transaction Signing**. Last participant that will decrypt **encrypted
   outputs**, as a result of the process, will get a list of **outputs**
   (_without any encryption_) and will send them to **service**. **Service**
   will form a transaction and send it to each participant for signing.
6. **Transaction Verification**. Each participant will verify that the transaction
   is valid and contains their **intputs** and **outputs**, and them will sign it.
7. **Transcation Sending**. After receving all required signatures, **service**
   will send the transaction to the **Ethereum** network.
