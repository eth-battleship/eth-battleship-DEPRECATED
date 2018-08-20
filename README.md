## blockchain-battleship

[![Build Status](https://travis-ci.org/eth-battleship/eth-battleship.github.io.svg?branch=source)](https://travis-ci.org/eth-battleship/eth-battleship.github.io)

Live demo (Ropsten): [https://eth-battleship.github.io/](https://eth-battleship.github.io/)

## Developer guide

Install [Truffle](https://truffleframework.com/docs/getting_started/installation):

```shell
npm i -g truffle
```

In the project folder, prepare the contracts:

```shell
npm install
truffle install zeppelin
truffle compile
```

Now let's get an Ethereum client running and connected to one of the test networks.
To run a local development chain do:

```
truffle develop
```

Now copy `truffle-config.js` to `truffle.js` and edit the contents according
to which network you wish to deploy to. Then run:

```
truffle migrate
```

Now you can the tests to ensure everything works:

```shell
truffle test
```

To run the app (against locally running dev chain):

```shell
./dev
```

## Tech architecture

The on-chain contract is used for "settlement" only - all moves are recorded off-chain
to ensure a good user experience (i.e. no delays and no need to make expensive
contract calls whilst game is in progress).

**Authentication**

Off-chain data is currently stored in Google Firestore cloud - I chose this
as it was easy and quick to get started with compared to IPFS/Swarm, plus it
provides real-time push updates built-in, meaning the game UI can be updated
in real-time as and when events take place. For v2 I would look into using
more decentralized alternatives and writing my own real-time notificatin system.

An individual
player's boards and moves data for a given game is stored in document only
accessible by that player. The way this works is by asking players to "sign in" prior to playing. This process
involves calling `web3.eth.sign` to get the player to sign a string. Part of this
generated signature (which is unique to the players's ETH account and can only be
generated by the owner of the private key of said account) is used as an
_authentication key_ of sorts for the player, allowing us to store data in the
cloud that can only be accessed by them.

In the cloud db we have a collection called _playerData_ which internally maps
`{game id}-{player auth key}` to the player's board and moves for that
particular game (note that `game id` is basically the game's deployed contract
address). This db collection does not permit listing, meaning you have to the
id of the document you wish to fetch, thus preventing other player's from
reading your data.

**Known issue: player DoS**

Because both players can write to the game data, it would be possible for one
player1 to remove the other player's address from the game data blob. The other
player would still be able to view the game in the "all games" list, but no
longer in the "my games" list.

This could easily be mitigated by having a separate db collection which maps
player _authentication key_ to game id - we would then populate the "my games"
list from this collection. Obviously we would forbid the ability to list the
contents of this db collection.

**Known issue: hit calculation**

At present each player calculates and stores the hits for their oppponent, i.e.
Player2 will check Player1's moves against their board and report back any hits.
Thus we rely on each player to be honest about reporting their opponent's hits
or misses, which is obviously not a good thing!

We could have a backend server which checks player hits, but then we'd have to
trust the server. What we need is a way to trust result of a computation of a
hit from the opponent player, perhaps what we need is a [zkSnark](https://blog.ethereum.org/2016/12/05/zksnarks-in-a-nutshell/)! However,
due to time limitations I haven't been able to implement this for the game. For
now, players are assumed not to be malicious - this is also why betting has not
yet been implemented for the game.
