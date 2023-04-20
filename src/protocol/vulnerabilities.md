# Drawbacks

## Coordinator

In this implementation, **service** is a trusted server, that coordinates
participants. Although the **coordinator** is not able to reveal the private
information of the participants, it can still prevent the protocol from starting
or completing.

A possible solution is to make an open, decentralized implementation of **service**,
that can be run and trusted by anyone.

## Sybil attack

Participants can collaborate in the room, to reveal the identities of other
participants.

For example, if 4 of 5 participants are malicious, they can reveal the identity
of the last participant, by revealing their outputs to each other.

A possible solution is to make a much larger number of participants in shuffling
process, so that the probability of a such attack is very low.

The other solution is to make an unpredictable, random algorithm that will distribute
participants into smaller "rooms" which will shuffle separately.

## Timing attacks

The protocol implementation relies on network communication with its latencies
and possible disconnections during the protocol execution. That's why one of the
participants can repeatably disrupt the protocol execution, by disconnecting.

## Sudoku analysis

While Coin Shuffle provides a reasonable level of privacy, users still need to
be careful about not revealing their identity through their actions. For
example, if the user will send his tokens to the same address from different
shuffling processes, some external viewers can easily analyze the history of the
transactions and participants of the recent shuffling, and reveal the identity
of the user by Sudoku analysis or even just intercept inputs' and outputs'
addresses of the transactions.

The only solution is to use new separate output addresses for each
shuffling process.
