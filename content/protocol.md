---
title: "Protocol"
date: 2023-04-14T19:08:56+03:00
draft: false
---

# Description

## Introduction

The **Coin Shuffle** protocol tries to solve the problem of **privacy** in the
**Ethereum** network by providing a **trustless** way to shuffle tokens between
multiple addresses in a way that the **history** of the shuffling is
**untraceable**.

## Overview

First of all, protocol reaches that goal by converting standard **Ethereum**
balance system to a **UTXO** system. This is done by creating a **smart
contract** that holds the **UTXO**s. Each **UTXO** is a prove of ownership of
some amount of **ERC20** tokens on the **Ethereum** network.

**Coin Shuffle** protocol implementation uses this **smart contract** to
**shuffle** the **UTXO**s between multiple addresses in one transaction. For that
purpose, the **smart contract** provides a function that takes a list of
**UTXO**s, a list of **addresses**, and proves that the **UTXO**s can be spent
by one of the **addresses**.

The contract spends old **UTXO**s and creates new **UTXO**s in batch for each of
**addresses**. So, if all input **UTXO**s has the same **ERC20** tokens
and **amount** then the output **UTXO**s will have the same **ERC20** tokens and
**amount**, so there will be no way to identify for each **input** from batch which
**output** belongs to it.

But how users can collaborate to create such transaction without revealing their
**output** addresses to each other? - Out answer is **Coin Shuffle** protocol.

## Coin Shuffle Protocol

For further explanation, we will use the following terms:

+ **Input** - a UTXO that is spent by the smart contract.
+ **Output** - an address that will receive a UTXO after shuffling.
+ **Encoded Outputs** - a list of **outputs** encoded using RSA secret keys. 
+ **Participant** - a user that wants to shuffle his UTXOs.
+ **Service** - a service that provides **coordination** logic for participants
    without any ability to reveal **output** addresses or steal **UTXO**s.
+ **Room** - a group of participants that are currently shuffling their **UTXO**s.

The shuffling process is done in the following steps:

1. **Participants Registration**. Participants register in the **service** by providing
    their proof of ownership of **UTXO**s that they want to shuffle.
2. **Participants connection**. Participants send their RSA public keys to the
   **service** to notify service that they are ready for shuffling.
3. **Keys distribution** - **service** defines order of shuffling and sends RSA
   public keys that are required for **outputs** **encoding** and **deconding**
   to each participant.
4. **Shuffling**. **Service** sends **encoded outputs** to each participant,
   participants partially **decode** them (_with keys that they have_),
   **encode** their **output** and send them to **service** which will
   pass them to next participant.
5. **Transaction Signing**. Last participant that will decode encoded outputs,
   as a result of the process, will get a list of **outputs** (_without any encryption_)
   and will send them to **service**. **Service** will form a transaction and 
   send it to each participant for signing.
6. **Transaction Verification**. Each participant will verify that the transaction
   is valid and contains their **intputs** and **outputs**, and them will sign it.
7. **Transcation Sending**. After receving all required signatures, **service**
   will send the transaction to the **Ethereum** network.

## Practical Example

Assume we have 5 participants that want to shuffle their **UTXO**s. Each
participant has a **UTXO** with the same **ERC20** tokens and **amount**, for
example 5 USDT tokens, and they want to shuffle them to 5 unknown addresses.

Lets name them **Alice**, **Bob**, **Charlie**, **David**, and **Eve**:

```goat
Alice    Bob   Charlie  David  Eve
  +       +       +       +     +
+-+-+   +-+-+   +-+-+   +-+-+ +-+-+
|5 $|   |5 $|   |5 $|   |5 $| |5 $|
+-+-+   +-+-+   +-+-+   +-+-+ +-+-+
   \_____  \     /   ____/  ____|
         \  |   |   |      /
          +-+---+---+---+-'
          |             |
          +-+--+--+-+-+-+
            |  |  | | |
            ?  ?  ? ? ?
```

**Alice**, **Bob**, **Charlie**, **David**, and **Eve** want to send their
UTXOs to some 5 addresses, and they don't want to reveal receivers of that UTXOs
to anybody. So, for coordination, they will use a **service** which implements
**Coin Shuffle** protocol.

```goat
Alice    Bob   Charlie  David  Eve
  +       +       +       +     +
   \_____  \     /   ____/  ____|
         \  |   |   |      /
          +-+---+---+---+-'
          | Coordinator |
          +-------------+
```

**Alice**, **Bob**, **Charlie**, **David**, and **Eve** will register in the
**service** by providing their **UTXO**s with proof of ownership.

Service will organize them into **queue** until enough participants will be
registered.

> In this example, we will assume that 5 participants are enough to shuffle.

```goat
Alice     Bob    Charlie   David    Eve
  +        +        +        +       +
.-+--.   .-+--.   .-+--.   +-+--+  +-+--+
|5$  |   |5$  |   |5$  |   |5$  |  |5$  |
|  @A|   |  @B|   |  @C|   |  @D|  |  @E|
'-+--'   '-+--'   '--+-'   +-+--+  +-+--+
   \        \     __/   ____/   ____/
    \        |   |     /       /
     '----+--+---+----+-+-----'
          | Coordinator |
          +-+--+--+-+-+-+
                      +-----+ .--. .--. .--. .--. .--.
                      |queue|-|@E|-|@C|-|@B|-|@D|-|@A|
                      +--+--+ '--' '--' '--' '--' '--'
```

Then, when enough participants will be registered, **service** will form a **room**,
and notify about that participants, so they can send their RSA public keys.
  
```goat
Alice    Bob   Charlie  David  Eve
  +       +       +       +     +
  '---<-.  \     /  .->---' .->-'
         \  ^   ^   |      /
          +-+---+---+---+-'
          | Coordinator |
          +--+----------+
           .-+-------------.
           |Room:          |
           | @A,@B,@C,@D,@E|
           '---------------'

```

As the response, paritipants will send their RSA public keys to the **service**:

> OK{#} - is a RSA public key of participant.

```goat
  Alice     Bob    Charlie   David   Eve
+-+---+   +-+---+  +-+---+   +-+---+ +-+---+
|(A)pk|   |(B)pk|  |(C)pk|   |(D)pk| |(E)pk|
+-+---+   +-+---+  +-+---+   +-+---+ +-+---+
  |         |     ___|         |       |
  '--->-.   |    /    .---<----'     .-'
         \  v   v     |   .----<-----'
          +-----------+-+-'
          | Room        |
          +-------------+
  
```

**Service** will define order of shuffling and send RSA public keys that are
required for **outputs** **encoding** and **deconding** to each participant:

```goat
Alice   Bob    Charlie David   Eve
  ^      ^       ^       ^      ^
.----. .----.  .----.  .----. .----.
|(B) | |(C) |  |(D) |  |(E) | '-+--'
|(C) | |(D) |  |(E) |  |  pk|   |
|(D) | |(E) |  |  pk|  '-+--'   |
|(E) | |  pk|  '-+--'    |      |
|  pk| '-+--'    |       |      |
'-+--'   |_      |      _|      |
  |        \     |     /        |
  '-------+-+----+----+-+-------'
          | Room        |
          +-------------+
```

Then **service** will send **encoded outputs** to each participant. starting
with **Alice**:

> Alice is the first participant in the **room**, so she will just encode her **output**.

Table below shows which keys which participant have to encode their **outputs**:

| Key / Participant | Alice | Bob | Charlie | David | Eve |
|-------------------|-------|-----|---------|-------|-----|
| (A)pk             | ✅    | ✅  | ✅      | ✅    | ✅  |
| (B)pk             | ❌    | ✅  | ✅      | ✅    | ✅  |
| (C)pk             | ❌    | ❌  | ✅      | ✅    | ✅  |
| (D)pk             | ❌    | ❌  | ❌      | ✅    | ✅  |
| (E)pk             | ❌    | ❌  | ❌      | ❌    | ✅  |

Firstly, **Alice** will encode her **output** with her public keys:

```goat
Alice:
                                   B.-------.
                      C.-----.     |C.-----. |
          D.---.      |D.---. |    ||D.---. ||
 E.-.     |E.-. |     ||E.-. ||    |||E.-. |||
 | A | -> || A ||  -> ||| A ||| -> |||| A ||||
  '-'     | '-' |     || '-' ||    ||| '-' |||
           '---'      | '---' |    || '---' ||
                       '-----'     | '-----' |
                                    '-------' 
```

Then, **Alice** will send **encoded output** to **Bob** through **service**.
**Bob** know, that received **encoded output** is encoded with his public key,
so he will decode it, then encode his **output** with his public keys, and then
send it to **Charlie**:


```goat
Bob decodes Alice output with his secret key:

B.-------.    
|C.-----. |    C.-----. 
||D.---. ||    |D.---. |
|||E.-. |||    ||E.-. ||
|||| A |||| -> ||| A |||
||| '-' |||    || '-' ||
|| '---' ||    | '---' |
| '-----' |     '-----' 
 '-------' 

Bob encodes his outputs with his public keys:
                      C.-----.
          D.---.      |D.---. | 
 E.-.     |E.-. |     ||E.-. || 
 | B | -> || B ||  -> ||| B ||| 
  '-'     | '-' |     || '-' || 
           '---'      | '---' | 
                       '-----'                             
Bobs encoded outputs:

C.-----.    C.-----.  
|D.---. |   |D.---. |
||E.-. ||   ||E.-. ||
||| A ||| + ||| B |||
|| '-' ||   || '-' ||
| '---' |   | '---' |
 '-----'     '-----' 
```

**Charlie** will decode it with his public key and encode his **output**:

```goat
Charlie decodes Bob's encoded outputs with her secret key:

C.-----.   
|D.---. |   D.---.
||E.-. ||   |E.-. |
||| A ||| ->|| A ||  
|| '-' ||   | '-' | 
| '---' |    '---'
 '-----' 
 
 C.-----.    
|D.---. |    D.---. 
||E.-. ||    |E.-. |
||| B ||| -> || B ||
|| '-' ||    | '-' |
| '---' |     '---' 
 '-----' 

Charlie encodes her outputs with her public keys:

          D.---.
 E.-.     |E.-. |
 | C | -> || C ||
  '-'     | '-' |
           '---'

Charlie's encoded outputs:

D.---.    D.---.    D.---.
|E.-. |   |E.-. |   |E.-. |
|| A || + || B || + || C ||
| '-' |   | '-' |   | '-' |
 '---'     '---'     '---'
```
