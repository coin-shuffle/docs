# Overview

For further explanation, we will use the following terms:

+ **Participant** - a user that wants to shuffle his UTXOs.
+ **Input** - a UTXO that the participant wants to shuffle, spent.
+ **Output** - an address that will receive a UTXO after shuffling.
+ **Encrypted Outputs** - a list of **outputs** encrypted by RSA secret key. 
+ **Service** - a service that provides **coordination** logic for participants
    without any ability to reveal **output** addresses or steal **UTXO**s.
+ **Room** - a group of participants that are currently shuffling their **UTXO**s.

The shuffling process is done in the following steps:

1. **Participants Registration**. Participants register in the **service** by providing
    their ownership proof of **UTXO**s that they want to spend.
1. **Shuffle start**. **Service** waits for enough participants to register and
    then starts the shuffling process.
1. **Participants connection**. Participants send their RSA public keys to the
   **service** to notify service that they are ready for shuffling.
1. **Keys distribution** - **service** defines order of shuffling and sends RSA
   public keys that are required for participants to decrypt **encrypted
   outputs**.
1. **Shuffling**.
   1. First participant encrypts his **output** with the RSA public keys of the next
      participants, in order, that each next participant will be able to decrypt
      its upper "layer" of encryption. Then, the participant sends **encrypted
      output** to the **service**.
   1. Next participant decrypts **encrypted outputs** of the previous
      participant and encrypts his **output** with RSA public keys of next
      participants and sends new **encrypted outputs** to the **service**.
   1. This process continues until the last participant will get a list of
      **outputs** without any encryption.
1. **Transaction distribution**. After the last participant sends a fully decrypted
   outputs to the **service**, **service** forms a transaction, that each
   the participant should sign.
1. **Transaction signing**. Each participant verifies a transaction,
   sees that has **input** and **output** are included, signs it, and sends
   signature to the **service**.
1. **Transaction sending**. **Service** gathers all signatures and sends
   transaction to the network.
