# Practical Example

Assume we have 5 participants that want to shuffle their **UTXO**s. Each
participant has a **UTXO** with the same **ERC20** tokens and **amount**, for
example 5 USDT tokens, and they want to shuffle them to 5 unknown addresses.

Lets name them **Alice**, **Bob**, **Charlie**, **David**, and **Eve**:

```mermaid
flowchart TB
    Alice
    Bob
    Charlie
    David
    Eve
```

**Alice**, **Bob**, **Charlie**, **David**, and **Eve** want to send their
UTXOs to some 5 addresses, and they don't want to reveal receivers of that UTXOs
to anybody. So, for coordination, they will use a **service** which implements
**Coin Shuffle** protocol.

```mermaid
flowchart TB
    Alice --- Service
    Bob --- Service
    Charlie --- Service
    David --- Service
    Eve --- Service
```

**Alice**, **Bob**, **Charlie**, **David**, and **Eve** will register in the
**service** by providing their **UTXO**s with proof of ownership.

Service will organize them into **queue** until enough participants will be
registered.

> In this example, we will assume that 5 participants are enough to shuffle.
```mermaid
flowchart TB
    Alice --> alice_utxo(5$A + Alice's signature)
    Bob --> bob_utxo(5$B + Bob's signature)
    Charlie --> charlie_utxo(5$C + Charlie's signature)
    David --> david_utxo(5$D + David's signature)
    Eve --> eve_utxo(5$E + Eve's signature)
    alice_utxo --> Service
    bob_utxo --> Service
    charlie_utxo --> Service
    david_utxo --> Service
    eve_utxo --> Service
    
    Service --> queue[[Queue: E, A, D, B, C]]
```

Then, when enough participants will be registered, **service** will form a **room**,
and notify about that participants, so they can send their RSA public keys.

```mermaid
flowchart TB
    Service --> room[[Room: A, B, C, D, E]]
```

As the response, paritipants will send their RSA public keys to the **service**:

```mermaid
flowchart TB
    Alice --> alice_pk(Alice's RSA public key)
    Bob --> bob_pk(Bob's RSA public key)
    Charlie --> charlie_pk(Charlie's RSA public key)
    David --> david_pk(David's RSA public key)
    Eve --> eve_pk(Eve's RSA public key)
    alice_pk --> Service
    bob_pk --> Service
    charlie_pk --> Service
    david_pk --> Service
    eve_pk --> Service
```

**Service** will define order of shuffling and send RSA public keys that are
required for **outputs** **encoding** and **decoding** to each participant:

```mermaid
flowchart TB
    Service --> alice_keys[[Alice's keys: B, C, D, E]]
    Service --> bob_keys[[Bob's keys: C, D, E]]
    Service --> charlie_keys[[Charlie's keys: D, E]]
    Service --> david_keys[[David's keys: E]]
    Service --> eve_keys[[Eve's keys:]]
    alice_keys --> Alice
    bob_keys --> Bob
    charlie_keys --> Charlie
    david_keys --> David
    eve_keys --> Eve
```

> Note, that each participant alread knows his own public key.

Table of **keys** that each participant has:

| Key / Participant | Alice | Bob | Charlie | David | Eve |
|-------------------|-------|-----|---------|-------|-----|
| Alice             | ✅    | ✅  | ✅      | ✅    | ✅  |
| Bob               | ❌    | ✅  | ✅      | ✅    | ✅  |
| Charlie           | ❌    | ❌  | ✅      | ✅    | ✅  |
| David             | ❌    | ❌  | ❌      | ✅    | ✅  |
| Eve               | ❌    | ❌  | ❌      | ❌    | ✅  |

Then **service** will send **encrypted outputs** to each participant, starting
with **Alice**:

> Alice is the first participant in the **room**, so she will receive list of
> empty encrypted **inputs**, and  encrypt her **output**.

Firstly, **Alice** will encrypt her **output** with her public keys:

```mermaid
flowchart LR
    alice_output["@A"]-- Eve's key -->alice_output1["E[ @A ]"]
    alice_output1-- David's key -->alice_output2["D[ E[ @A ] ]"]
    alice_output2-- Charlie's key -->alice_output3["C[ D[ E[ @A ] ] ]"]
    alice_output3-- Bob's key -->alice_output4["B[ C[ D[ E[ @A ] ] ] ]"]
```

Then, **Alice** will send **encrypted output** to **Bob** through **service**.
**Bob** knows, that received **encrypted output** is encrypted with his RSA
public key, so he will decrypt it using his secret key:


```mermaid
flowchart LR
    alice_output3["B[ C[ D[ E[ @A ] ] ] ]"]-- Bob's secret key -->bob_output1["C[ D[ E[ @A ] ] ]"]
```

Then **Bob** will encrypt his **output** with his public keys:

```mermaid
flowchart LR
    bob_output["@B"] -- Eve's key -->bob_output1["E[ @B ]"]
    bob_output1 -- David's key -->bob_output2["D[ E[ @B ] ]"]
    bob_output2 -- Charlie's key -->bob_output3["C[ D[ E[ @B ] ] ]"]
```

Then, **Bob** will shuffle them, and  send **encrypted outputs** to **Charlie** through **service**.

```mermaid
flowchart LR
    Bob --- bobs_outputs[["C[ D[ E[ @B ] ] ] <br> C[ D[ E[ @A ] ] ]"]]
    bobs_outputs --> Service
    Service --> Charlie
```

**Charlie** will decrypt it's upper layers with her secret key:

```mermaid
flowchart LR
    bobs_outputs["C[ D[ E[ @A ] ] ]<br> C[ D[ E[ @B ] ] ]"]-- Charlie's secret key -->charlie_output1["D[ E[ @A ] ]<br> D[ E[ @B ] ]"]
```

Then **Charlie** will encrypt her **output** with her public keys:

```mermaid
flowchart LR
    charlie_output["@C"] -- Eve's key -->charlie_output1["E[ @C ]"]
    charlie_output1 -- David's key -->charlie_output2["D[ E[ @C ] ]"]
```

Shuffle them, and send to **David**:

```mermaid
flowchart LR
    Charlie --- charlies_outputs[["D[ E[ @A ] ] <br> D[ E[ @C ] ] <br> D[ E[ @B ] ]"]]
    charlies_outputs --> Service
    Service --> David
```

The process will continue until **Eve** will receive **encrypted outputs** from
**David**. **Eve** will finally get fully decrypted **outputs** from all
participants:

```mermaid
flowchart LR
    davids_outputs["E[ @C ]<br> E[ @D ]<br> E[ @A ]<br> E[ @B ]"]-- Eve's secret key -->eve_output1["@C<br> @D<br> @A<br> @B"]
```

**Eve** will send that **outputs** to **service**, **service** will
form transaction with all **inputs** and **outputs**, and send it to
all participants for signing.

```mermaid
flowchart TB
    Service --> transaction[["Transaction. <br> Inputs: $5D, $5C, $5B, $5A, $5E <br> Outputs: @C, @D, @A, @B, @E"]]
    transaction --> Alice
    transaction --> Bob
    transaction --> Charlie
    transaction --> David
    transaction --> Eve
```

Each participant will see, that transaction is valid and at least contains
their input and output, so they will sign it and send it back to **service**.

The **service** will gather all required signatures, and then send it to the network.
