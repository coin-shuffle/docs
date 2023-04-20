# Introduction

The **Coin Shuffle** protocol tries to solve the problem of **privacy** in the
decentralized network by providing a **trustless** way to shuffle tokens between
multiple addresses in a way that the **history** of the shuffling is
**untraceable**.

Protocol is based on **UTXO** (unspent transaction output) model (which is used,
for example, in Bitcoin). In this model, each token is represented by a
**transaction output** - a proof that a certain amount of tokens was sent to a
certain address (usually, a hash of the transaction in which UTXO was created).

The **Coin Shuffle** protocol enables a group of users to combine their unspent
transaction outputs (UTXOs) and shuffle them in a manner that ensures the
resulting tokens are sent to different addresses in batch, while keeping the
association between the original **inputs** and the new **outputs**
confidential.
